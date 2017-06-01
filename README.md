# What is this?

The main content of this repository is a Vagrantfile, which provides you a virtual machine with an Ubuntu Linux (Xenial), a Neo4j (> 3.0), an Asciidoctor setup, and a webserver serving your own Neo4j playbooks, i.e., tutorials.


## Getting started...

  * If not already installed on your machine setup VirtualBox (http://virtualbox.org) and Vagrant (https://www.vagrantup.com/intro/getting-started/install.html).
  * Clone this repository

```
git clone git@github.com:HelgeCPH/neo4j-training.git
```


## Neo4j


After running `vagrant up`, the Neo4j database server is up and running. You can access the webclient on your host machine on http://127.0.0.1:7474/browser/.

The standard login is `neo4j` and the standard password is `demo`.


## Creating your own tutorials

Tutorials are written as Asciidoctor documents. You can edit those either on your host machine `` or on the virtual machine ``.

The Asciidoctor setup is in essence just the one given on https://github.com/neo4j-contrib/developer-resources/blob/gh-pages/resources/guide-create-neo4j-browser-guide/guide-create-neo4j-browser-guide.adoc

```
vagrant ssh
suplemon /synced_folder/adoc/tutorial_example.adoc
cd neo4j-guides/

./run.sh /synced_folder/adoc/tutorial_example.adoc /synced_folder/html/tutorial_example.html

python3 serve_playbooks.py
```


