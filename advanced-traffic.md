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

# Advanced Traffic Management
This section discusses advanced traffic management using the load balancer.

### Max Connections
Use the ‘max connections’ configuration under the VIP port to limit the maximum number of concurrent connections possible for a given VIP port. By default, no limit is specified. The maximum possible concurrent connections against an individual VIP port or across all VIP ports is 64,000. 

### Connection Timeouts
There are three internal system timeout settings. These settings cannot be customized. 

* *** Timeout for Server-side Connection Attempt (in seconds) – 5 seconds:** The maximum time window that the load balancer can use to establish TCP connection with the back-end server. If the connection attempt is unsuccessful, the load balancer will try the next available server, according to the load balancing method you've configured. 

* *** Client-side Idle Connection Timeout (in seconds) – 50 seconds:** The maximum idle time, after which the load balancer  brings down the client-side connection, if the client has failed to close its connection properly. 

* *** Server-side Idle Connection Timeout (in seconds) – 50 seconds:** The maximum idle time (with back-end protocol configuration of TCP), after which the load balancer closes the server-side connection. With the back-end protocol configuration of HTTP, if the load balancer fails to receive a response to its HTTP request within the idle timeout window, it will return an error message to the end-client. 

### HTTP Keep Alive
The load balancer supports `HTTP keep alive` as long as it is enabled on both the client and back-end servers. The load balancer attempts to re-use the server-side HTTP connections to increase connection efficiency and reduce latency.
