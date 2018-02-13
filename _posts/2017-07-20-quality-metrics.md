---
layout: page
title: "Quality Metrics"
category: ext
date: 2017-06-20 15:24:20
order: 1
---

This page contains quality metrics for AGORA. Developers should check this to make sure
that their code complies with the defined standards and that the service is clean and fast.

### Code Quality
To test whether the code complies with coding standards, you may use linting tools for each
repository. The procedure for each of the three repositories is outlined below:
<ul>
<li><a target="_blank" href="https://github.com/AuthEceSoftEng/agora-elasticsearch-client">AGORA Elasticsearch Client</a>: Download and install <a target="_blank" href="https://www.pylint.org/">Pylint</a> and issue the command `pylint main.py dbmanager.py libs` in the root of the repo</li>
<li><a target="_blank" href="https://github.com/AuthEceSoftEng/agora-ast-parser">AGORA AST Parser</a>: Download and install <a target="_blank" href="https://pmd.github.io/">PMD</a> and issue the command `pmd -d src -R java-basic.xml` in the root of the repo</li>
<li><a target="_blank" href="https://github.com/AuthEceSoftEng/agora-web-applicationr">AGORA Web Application</a>: Download and install <a target="_blank" href="https://eslint.org/">ESLint</a> and issue the command `eslint js` in the root of the repo</li>
</ul>

An indicative run of the above tools shows that the AGORA AST Parser does not have any issues, while the AGORA Web Application and 
the AGORA Elasticsearch Client are also of high linting quality, with the latter achieving linting score 8.85/10.

### Service Performance

#### Response Time
To test the response time of the service, one can measure how much does it take to complete a query.
As the main point of assessment is the performance of the index, the measurement is performed using
API queries (the ones available at <a target="_blank" href="http://agora.ee.auth.gr/api/">http://agora.ee.auth.gr/api/</a>).
One can perform the measurement by running the following python script:
```python
import time
import requests

queries = [
	'{"query":{"bool":{"should":[{"match":{"_all":"filereader"}},{"match":{"extension":"java"}}]}}}',
	'{"query":{"bool":{"should":[{"match":{"code.class.name":"Stack"}},{"nested":{"path":"code.class.methods","query":{"bool":{"should":[{"match":{"code.class.methods.name":"push"}},{"term":{"code.class.methods.returntype":"void"}}]}}}},{"nested":{"path":"code.class.methods","query":{"bool":{"should":[{"match":{"code.class.methods.name":"pop"}},{"term":{"code.class.methods.returntype":"int"}}]}}}}]}}}',
	'{"query":{"bool":{"should":[{"match":{"files.code.class.name":"Export"}},{"match":{"files.code.class.extends":"WizardPage"}},{"match":{"files.code.imports":"eclipse"}}]}}}',
	'{"query":{"bool":{"should":[{"has_child":{"type":"files","query":{"match":{"code.class.implements":"Model"}}}},{"has_child":{"type":"files","query":{"match":{"code.class.implements":"View"}}}},{"has_child":{"type":"files","query":{"match":{"code.class.implements":"Controller"}}}},{"has_child":{"type":"files","query":{"match":{"code.class.extends":"JFrame"}}}}]}}}',
	'{"query":{"match":{"analyzedcontent":"File myfile = new File(\\"myfile.xml\\");\\nDocumentBuilderFactory dbFactory = DocumentBuilderFactory.newInstance();\\nDocumentBuilder dBuilder = dbFactory.newDocumentBuilder();\\nDocument doc = dBuilder.parse(myfile);"}}}'
]

repetitions = 5

total_time = 0
for i, query in enumerate(queries):
	query_times = []
	for repetition in range(repetitions):
		time_start = time.time()
		r = requests.post('http://agora.ee.auth.gr:8080/_search', data = query)
		if r.ok:
			time_elapsed = time.time() - time_start
			query_times.append(time_elapsed)
		else:
			print("Error on query %d" %(i+1))
			exit()
	avg_query_time = sum(query_times) / len(query_times)
	print("Average time per query for query %d: %.2f milliseconds" %(i + 1, avg_query_time))
	total_time += avg_query_time
print("Average time per query for all queries: %.2f milliseconds" %(total_time / len(queries)))
```

An example output for this command is given below:
```
Average time per query for query 1: 0.15 milliseconds
Average time per query for query 2: 0.63 milliseconds
Average time per query for query 3: 0.01 milliseconds
Average time per query for query 4: 0.02 milliseconds
Average time per query for query 5: 0.15 milliseconds
Average time per query for all queries: 0.19 milliseconds
```

#### Indexing Time
Measuring the indexing time requires instrumenting the
<a target="_blank" href="https://github.com/AuthEceSoftEng/agora-elasticsearch-client/blob/master/dbmanager.py#L90">add_project method of dbmanager</a> and the <a target="_blank" href="https://github.com/AuthEceSoftEng/agora-elasticsearch-client/blob/master/dbmanager.py#L90">add_projects method of dbmanager</a>.
The new add_project method should be the following:
```python
def add_project(self, project_address):
	"""
	Adds a project to the index or updates it if it already exists.
	
	:param project_address: the URL of the project to be added.
	"""
	project_id = '/'.join(project_address.split('/')[-2:])
	sys.stdout.write('\nDownloading project info for project ' + project_id)
	project, sourcefiles = self.gpdownloader.download_project(project_id)
	sys.stdout.write('. Done!\n')
	project_path = self.sourcecodedir + '/' + project['user'] + '/' + project['name']
	self.gitdownloader.git_pull_or_clone(project_id, project['git_url'], project_path, project['default_branch'])
	sys.stdout.write('Adding project to database!\n')
	sys.stdout.write('Compiling project')
	full_compiled_source = self.javaparser.parse_project(project_path)
	sys.stdout.write('. Done!\n')
	self.esclient.create_project(project)
	sys.stdout.write('Creating database entries')
	import time
	create_times = []
	for file_path, afile in self.get_enumerated_files_with_paths(project_path, sourcefiles):
		self.set_file_code_and_contents(file_path, afile, full_compiled_source)
		time_start = time.time()
		self.esclient.create_file(afile)
		create_times.append(time.time() - time_start)
	sys.stdout.write(' Done!\n')
	avg_create_time = sum(create_times) / len(create_times)
	sys.stdout.write("Average indexing time per file for project %s: %.3f milliseconds\n" %(project_id, avg_create_time))
	return avg_create_time
```
and the new add_projects method should be the following:
```python
def add_projects(self, project_addresses):
	"""
	Adds or updates a list of projects in the index.
	
	:param project_addresses: a list of project addresses to be added.
	"""
	total_time = 0
	for project_address in project_addresses:
		avg_create_time = self.add_project(project_address)
		total_time += avg_create_time
	print("Average indexing time per file for all projects: %.3f milliseconds" %(total_time / len(project_addresses)))
```
Note that the indexing time is measured for creating a project.
So, the projects should not be already created.

An example output for this analysis (executed on the <a target="_blank" href="https://github.com/AuthEceSoftEng/agora-elasticsearch-client/blob/master/projectlists/test_projects.txt">list of test projects</a> using the command `sudo python3 main.py add_projects ./projectlists/test_projects.txt`) is given below:
```
Average indexing time per file for project square/retrofit: 0.003 milliseconds
Average indexing time per file for project bumptech/glide: 0.004 milliseconds
Average indexing time per file for project JakeWharton/butterknife: 0.004 milliseconds
Average indexing time per file for project greenrobot/EventBus: 0.003 milliseconds
Average indexing time per file for project nostra13/Android-Universal-Image-Loader: 0.005 milliseconds
Average indexing time per file for project google/iosched: 0.004 milliseconds
Average indexing time per file for project square/picasso: 0.005 milliseconds
Average indexing time per file for project skylot/jadx: 0.004 milliseconds
Average indexing time per file for project alibaba/fastjson: 0.006 milliseconds
Average indexing time per file for project Netflix/Hystrix: 0.004 milliseconds
Average indexing time per file for all projects: 0.004 milliseconds
```

