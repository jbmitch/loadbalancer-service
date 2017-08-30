---

copyright:
  years: 2017
lastupdated: "2017-08-21"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# FAQs

This section contains answers to some frequently asked questions about IBM Bluemix Load Balancer Service.

## What's the maximum number of virtual ports I can define with my load balancer service?

While trying to create a new load balancer service, you may define up to two virtual ports. You can define additional virtual ports after the service is created. The maximum number of virtual ports allowed is 10. 

## Can I create an internal-only, private load balancer that can only be accessed by internal clients?  

Not at this time. The service hosted on the IBM Bluemix load balancer will be assigned a fully qualified domain name accessible from the public Internet. 

## What are the default settings and allowed values for various health check parameters?

The default settings and allowed values are listed below:

* **Health check interval:** Default is 5 seconds, range is 5 – 60 seconds
* **Health check response timeout:** Default is 2 seconds, range is 2 – 60 seconds
* **Max Retrials:** Default is 2 retrials, range is 1 to 10 retrials

**Note:** The health check response timeout value must always be less than the health check interval value. 

## Can I use compute instances residing in remote data centers with this service? 

It is recommended that your load balancer service and your compute instances reside locally within the same data center. The load balancer service’s graphical interface (GUI) will not show compute instances from other remote data centers. However, the GUI will include compute instances from other data centers within same city (for example, data centers whose names share the first three letters, such as `DALxx`). You can use the API interface to add compute instances from any remote data center. However, be aware that additional latency may result when accessing servers that are not local to your data center.  

## Which TLS version is supported with SSL offload? Which ciphers are supported?

The Bluemix Load Balancer Service supports TLS 1.2 with SSL termination. 

The following list details the supported ciphers (listed in order of precedence):  

* ECDHE-RSA-AES256-GCM-SHA384 
* ECDHE-RSA-AES256-SHA384 
* DHE-RSA-AES256-GCM-SHA384 
* DHE-RSA-AES256-SHA256 
* AES256-GCM-SHA384 
* AES256-SHA256 
* ECDHE-RSA-AES128-GCM-SHA256 
* ECDHE-RSA-AES128-SHA256 
* DHE-RSA-AES128-GCM-SHA256 
* DHE-RSA-AES128-SHA256 
* AES128-GCM-SHA256 
* AES128-SHA256 

## Can I customize my SSL cipher list?

Not at this time.

## Is there a limit to the maximum number of Load Balancer Service instances I can create within my account? 

Currently, you may create up to xx instances. If you need more instances, contact IBM Support. 


