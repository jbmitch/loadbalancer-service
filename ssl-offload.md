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

# SSL Offload
For HTTPS connections to VIP, the load balancer terminates incoming SSL connections and forwards plain-text HTTP connections to the back-end server. The back-end servers are thus offloaded from performing CPU-intensive SSL handshakes and encryption/decryption tasks, so they can use all their CPU cycles for processing application traffic. An SSL certificate is required for the load balancer to perform SSL offload tasks. You may use a pre-existing SSL certificate or purchase a new one, and manage it through the [Bluemix/Softlayer Certificate Store ](https://control.softlayer.com/security/sslcerts). 

**NOTE:** TLS version 1.2 is supported with SSL offload, while SSL v3 is not supported due to its vulnerabilities. The SSL cipher list is not customizable at this time. 
