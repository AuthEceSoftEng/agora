---
layout: default
title: "Home"
---

<div class="page-header">
  <h2> Home </h2>
</div>

### Get Started

This is the documentation page of AGORA, a Search Engine for Source Code Reuse.
In this website you may find all resources and manuals used to build, develop and run AGORA.
An online version of the service is available at:

<center><a target="_blank" href="http://agora.ee.auth.gr">http://agora.ee.auth.gr</a></center>

<p></p>

AGORA is currently in version 1.0 and has been sent to the SoftwareX journal. Upon review, the final version for the journal will be noted by an appropriate tag in all three AGORA repositories.

### About AGORA

AGORA is a Code Search Engine that offers advanced features in order to help developers find the source code that they need.

AGORA offers three types of search: Simple Search that searches the whole directory for the search terms provided, Advanced Search that provides syntax-aware search for source code, Snippet Search that allows searching by giving code snippets as search terms, and Project Search that allows searching for projects given class names, and other elements (implements and extends).

Additionally AGORA offers an open RESTful API for searching.

AGORA has been developed by Themistoklis Diamantopoulos under the supervision of Andreas Symeonidis in the <a target="_blank" href="https://issel.ee.auth.gr/">Intelligent Systems & Software Engineering Lab (ISSEL)</a> of the <a target="_blank" href="http://ee.auth.gr/en/">School of Electrical & Computer Engineering</a> of the Aristotle University of Thessaloniki. For any questions or suggestions, feel free to contact us at the email agora[at]olympus[dot]ee[dot]auth[dot]gr 

### AGORA Documentation

The main repository of AGORA that keeps the links to all repositories and holds the versions of the service is
available at <a target="_blank" href="https://github.com/AuthEceSoftEng/agora">https://github.com/AuthEceSoftEng/agora</a>.

AGORA comprises three modules residing in three different repositories:
<ul>
<li>an AST Parser for Java source code, available at <a target="_blank" href="https://github.com/AuthEceSoftEng/agora-ast-parser">https://github.com/AuthEceSoftEng/agora-ast-parser</a></li>
<li>an Elasticsearch Client for the AGORA index, available at <a target="_blank" href="https://github.com/AuthEceSoftEng/agora-elasticsearch-client">https://github.com/AuthEceSoftEng/agora-elasticsearch-client</a>, and</li>
<li>a web application that can be used to issue queries, available at <a target="_blank" href="https://github.com/AuthEceSoftEng/agora-web-application">https://github.com/AuthEceSoftEng/agora-web-application</a>.</li>
</ul>

The links to the left provide instructions for installing AGORA, as well as documentation references for each of the individual modules.

