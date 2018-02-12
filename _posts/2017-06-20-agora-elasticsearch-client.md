---
layout: page
title: "AGORA Elasticsearch Client"
category: ref
date: 2017-06-20 15:24:20
order: 1
---

The Elasticsearch client of AGORA allows creating and deleting the AGORA index, adding, removing
and updating projects, and performing flushand backup operations on the index. It is built in
Python and all dependencies can be installed issuing the command:

```
sudo pip3 install -r requirements.txt
```

The code is available at <a target="_blank" href="https://github.com/AuthEceSoftEng/agora-elasticsearch-client">https://github.com/AuthEceSoftEng/agora-elasticsearch-client</a>

To execute the tool you must have set up the Elasticsearch service (see the <a href="/doc/installation-instructions">Installation Instructions</a>) and then set the options in file agora.properties.
The structure of the file is the following:

```
[AGORAProperties]

### ASTParser path
ASTParserPath = (path to agora-ast-parser.jar)

### Git system command
gitcommand = (path to git executable)

### GitHub credentials
GitHubUsername = (username of GitHub account)
GitHubPassword = (password of GitHub account)

### AGORA credentials
AGORAUsername = (username of AGORA admin account)
AGORAPassword = (password of AGORA admin account)

### Index options
indexname = agora
host = localhost
port = 9200
sourcecodedir = (location where the cloned code will be stored)
backupdir = (location where the backup of AGORA will be stored)
```

The username and the password of the AGORA admin account are the ones that were set when configuring apache. The AST parser path should be `/home/USERNAME/agora-ast-parser/target/agora-ast-parser-0.1.jar`, while the source code directory and the backup directory are the ones that were set at the prerequisites (`/home/USERNAME/elasticsearch-5.2.0/code` and `/home/USERNAME/elasticsearch-5.2.0/backup`), where USERNAME has to be changed to your own.