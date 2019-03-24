---
layout: post
title: Apache Solr Config API Insecure Deserialization (CVE-2019-0192)
---

Apache Solr Config API Insecure Deserialization (CVE-2019-0192)

Due to Insecure deserialization of requests from user, a vulnerability was reported in Apache Solr. A remote, unauthenticated attacker can exploit this vulnerability by sending a crafted HTTP request to the Config API. Successful exploitation could result in arbitrary code execution in the context of the server application. CVE-2019-0192 has been assigned to this vulnerability. The vulnerable versions are:
	Apache Software Foundation Solr 5.0.0 to 5.5.5
	Apache Software Foundation Solr 6.0.0 to 6.6.5 

Apache Solr is an open source search platform built on Java library called Lucene.
Apache Solr is popular search platform for web sites because it can index and search multiple sites and return recommendations for related content based on search queries taxonomy. It supports text search, hit highlighting, database integration. It also supports JSON REST API. It runs on port 8983.

Vulnerability is mainly due to insufficient validation of request to Config API.
Config API enables users to manipulate various aspects of solrconfig.xml file.
Solrconfig.xml file controls how Solr behaves by mapping request to different handlers. Solrconfig.xml contains most of the parameters for configuring Solr. It defines how search requests are managed and how to manipulate data for the user. 
Apache Solr is built on Java. Java allows serializing objects thus enabling them to be represented as a compact and portable byte stream. This byte stream can then be transferred over network and deserialized for use by the corresponding Java VM. 
Config API allows to configure Solr’s JMX server via HTTP POST request. By pointing it to malicious RMI server, an attacker could take advantage of Solr’s unsafe deserialization to trigger remote code execution on the Solr server.

Attacker can start the malicious RMI server by running below command:

![_config.yml]({{ site.baseurl }}/images/RMIServer.png)
 
In the above image, ysoserial payload with class JRMPListener is used to embed the command “touch /tmp/pwn.txt” which gets executed on Apache Solr when 

Attacker can then send the below POST request to Solr to set remote JMX server.
 
 ![_config.yml]({{ site.baseurl }}/images/POST.png)
 
Java Management Extensions (JMX) allow remote clients to connect to a Java Virtual Machine (JVM) and manage/monitor running applications in that JVM.
Management of application is done through MBeans. Using MBeans user can access and control the inner working of the application. This MBeans is accessed over different protocol called RMI(Java Remote Method Invocation).
The user who wants to use JMX/RMI interface on a server, creates JMXService URL(service:jmx:rmi:///jndi/rmi://<target system>:<port>/jmxrmi).

In this vulnerability, Attacker, Using POST request, sets the jmx.serviceUrl remotely via Config API using ‘set-property’ JSON object as shown below:

![_config.yml]({{ site.baseurl }}/images/jmx.png)
  
We should receive 500 error response from the Apache Solr with “undeclared checked exception; nested exception is” in the response body. It is shown in the below image:
 
 ![_config.yml]({{ site.baseurl }}/images/exception.png)

Due to improper validation, this jmx.serviceUrl can point to attacker controlled JMRP listener causing Apache Solr to initiate RMI connection to malicious JMRP Listener.
Now, Apache Solr will initiate 3-way handshake to malicious RMI server and sets up connection with malicious RMI server. 
This can be leveraged to perform RCE on the Apache Solr. One of the cases is, Attacker can send crafted serialized object.
The data transmission for the above vulnerability is shown below:

![_config.yml]({{ site.baseurl }}/images/Fina;.png)
 


