---
layout: page
title: "Installation Instructions"
category: doc
date: 2017-06-20 15:23:20
order: 1
---

This page provides installation instructions for setting up AGORA in Ubuntu Linux.

### Prerequisites
Install java and maven using the following commands (Oracle java is recommended):

```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
sudo apt-get install maven
```

Install python and pip using the following commands:

```
sudo apt-get install python3
sudo apt-get install python3-pip
```

Install bower using the following commands:

```
sudo apt-get install nodejs
sudo apt-get install npm
sudo npm install bower -g
sudo ln -s /usr/bin/nodejs /usr/bin/node
```

Install apache web server

```
sudo apt-get install apache2
```

Download and install elasticsearch (change USERNAME and USERGROUP to your own)

```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.2.0.tar.gz
tar -zxf elasticsearch-5.2.0.tar.gz
sudo chown -R USERNAME:USERGROUP elasticsearch-5.2.0/
mkdir elasticsearch-5.2.0/code
mkdir elasticsearch-5.2.0/backup
```

Git clone all repos into your home directory (or any other dir)

```
git clone https://github.com/AuthEceSoftEng/agora-elasticsearch-client.git
git clone https://github.com/AuthEceSoftEng/agora-ast-parser.git
git clone https://github.com/AuthEceSoftEng/agora-web-application.git
```

### Configurations

#### Configure elasticsearch

Step 1: use provided elasticsearch.yml configuration file (available also <a target="_blank" href="https://github.com/AuthEceSoftEng/agora-elasticsearch-client/blob/master/configuration/elasticsearch.yml">here</a>)

```
sudo cp agora-elasticsearch-client/configuration/elasticsearch.yml elasticsearch-5.2.0/config/elasticsearch.yml
```

Step 2: create scripts to start/stop the service

```
echo "./elasticsearch-5.2.0/bin/elasticsearch -d -p pid" > startElastic.sh
echo "kill \`cat pid\`" > stopElastic.sh
chmod 700 startElastic.sh
chmod 700 stopElastic.sh
```

Step 3: add the command `su USERNAME -c "/home/USERNAME/startElastic.sh"` in file `/etc/rc.local` (change USERNAME to your own)

Step 4: start the service using the command `./startElastic.sh`

#### Configure apache

Step 1: add `Listen 8080` in file `/etc/apache2/ports.conf`

Step 2: edit the provided agora-elasticsearch-client/configuration/agora.conf configuration file (available also <a target="_blank" href="https://github.com/AuthEceSoftEng/agora-elasticsearch-client/blob/master/configuration/agora.conf">here</a>) and set the IP the server and the administrator email twice, one time for each virtual host (in ports 80 and 8080)

Step 3: use the agora.conf configuration file

```
sudo mv /etc/apache2/sites-enabled/000-default.conf /etc/apache2/sites-enabled/000.default.conf.bak
sudo cp agora-elasticsearch-client/configuration/agora.conf /etc/apache2/sites-enabled/agora.conf
```

Step 3: enable mods

```
sudo a2enmod proxy
sudo a2enmod proxy_http
```

Step 4: set a password for the admin account

```
sudo mkdir -p /usr/local/apache/passwd/
sudo htpasswd -c /usr/local/apache/passwd/passwords admin
```

Step 5: restart the service

```
sudo service apache2 restart
```

### Build and Install AGORA

Build the agora-ast-parser

```
cd agora-ast-parser
mvn install
```

Install all requirements of the agora-elasticsearch-client

```
cd ../agora-elasticsearch-client
sudo pip3 install -r requirements.txt
```

Create a file agora.properties using the command `cp sample-agora.properties agora.properties` and fill it (according to the instructions provided <a target="_blank" href="/agora/ref/agora-elasticsearch-client">here</a>) with the appropriate paths, usernames and password. The username and the password of the AGORA admin account are the ones that were set at the Step 4 of configuring apache. The AST parser path should be `/home/USERNAME/agora-ast-parser/target/agora-ast-parser-0.1.jar`, while the source code directory and the backup directory are the ones that were set at the prerequisites (`/home/USERNAME/elasticsearch-5.2.0/code` and `/home/USERNAME/elasticsearch-5.2.0/backup`), where USERNAME has to be changed to your own.

Create and populate the index

```
sudo python3 main.py create_index
sudo python3 main.py add_projects ./projectlists/moststars.txt
```

Copy the contents of agora-web-application in apache web directory

```
sudo mv /var/www/html/index.html /var/www/
cd ../agora-web-application
sudo cp -r . /var/www/html/
```

Install all requirements of the agora-web-application

```
cd /var/www/html/
sudo bower install --allow-root
```

If you want to log all the queries, issue the following request on the service:

```
curl -XPUT 'http://localhost:9200/_all/_settings?preserve_existing=true' -d '{
   "index.search.slowlog.threshold.query.debug" : "0s",
   "index.search.slowlog.threshold.query.info" : "0s",
   "index.search.slowlog.threshold.query.trace" : "0s",
   "index.search.slowlog.threshold.query.warn" : "0s"
}'
```

Go to the localhost in the browser to check the website



