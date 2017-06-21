---
layout: page
title: "AGORA Elasticsearch client"
category: ref
date: 2017-06-20 15:24:20
order: 1
---

The Elasticsearch client of AGORA allows creating and deleting the AGORA index, adding, removing
and updating projects, and performing flushand backup operations on the index. It is built in
Python and all dependencies can be installed issuing the command:

```
pip install -r requirements.txt
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
