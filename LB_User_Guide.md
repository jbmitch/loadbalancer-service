# IBM Bluemix Load Balancer Service (Beta Release)

The IBM Bluemix Load Balancer service helps customers improve availability of their business-critical applications by distributing traffic among multiple application server instances and by forwarding traffic to healthy instancesonly.

## Table of Contents

 * [Welcome](#welcome)
 * [Getting Started](#getting-started)
 * [Overview of Features](#overview-of-features)
 * [Basic Load Balancing](#basic-load-balancing)
 * [Known Limitations](#known-limitations)
 * [Getting Support](#getting-support)
 * [Giving Feedback](#giving-feedback) 

## Welcome

This document describes features available with the IBM Bluemix Load Balancer (Beta) service. If you wish to sign-up for Beta service, or have any comments, send a note to [bluemixlbbeta@us.ibm.com](bluemixlbbeta@us.ibm.com). You will be provided with a URL link to access the Beta service.

## Getting Started

This section probably ought to show what the user would first see as a splash screen or other introductory screen, give a very general description of the offering, and show the user how to log in or otherwise start using the offering. It may include general information about how to navigate through the offering, and whatever other general information the writer knows to be relevant for new users.

## Overview of Features

This section might contain a summary, even a bullet-point summary, of key features of the offering. Without growing too lengthy, the section might give a basic description of what each feature does, and perhaps how to navigate to it. Sometimes it might make sense to combine "Getting Started" with "Overview of Features" in the document, but conceptually it is good to identify them separately, because they can be chunked differently in Bluemix.

Keeping the principle of "progressive disclosure" in mind, this section might give additional detail about items covered in the "Getting Started" section.

## Basic Load Balancing
The IBM Bluemix load balancer service can be used to distribute traffic among multiple server instances (bare metal and virtual server) residing locally within the same data center. 

### Public Load Balancer 
Each IBM Bluemix load balancer service is assigned one IPv4 Virtual IP (VIP). This VIP has a globally-unique IP address and can be accessed over public Internet. The backend compute servers (hosting your application) must be on an IBM/Softlayer private network. 

**NOTES:** 

* The VIP is not associated with any domain name. One may use an external domain service to map their custom, user-friendly domain name to this load balancer VIP address.

* As a good practice, it is recommended that your backend servers be provisioned as ‘private-only’, unless they require direct public connectivity. This helps achieve better security and preserves your public IP address as well. The applications hosted on these backend servers are still accessible over public network via load balancerVIP. 

### Front-end and Back-end Application Ports/Protocols
One may define up to four front-end application ports (protocols) under VIP and map them to respective ports (protocols) on the back-end application servers. The VIP and the front-end VIP ports are exposed to the external world. The incoming user requests are received on these ports. 

On the other hand, the back-end ports are only known internally. These back-end ports may or may not be the same as the front-end ports. As an example, the load balancer may be configured to receive incoming web/HTTP traffic on front-end port 80, while the back-end servers are listening on custom port 81. 

The supported front-end ports/protocols are HTTP, HTTPS and TCP. The supported back-end ports/protocols are HTTP and TCP. Incoming HTTPS traffic must be terminated at the load balancer to allow for plain-text HTTP communication with the backend server. 

**NOTES:**

* During the initial configuration, you can define up to two front-end ports only, even though the maximum supported port count is four. Once a load balancer is created, you can edit port configuration to define additional ports up to a maximum of four ports.

* All four VIP ports must map to the same set of back-end server instances

* The maximum number of server instances that may be placed behind a load balancer is 1000.

* The port range of 56500 to 56520 is reserved for management purposes and cannot be used under VIP.

###Load Balancing Methods
The following three load balancing methods are available for distributing traffic among back-end application servers:

* **Round Robin:** Round Robin is the default load balancing method. With this method, the load balancer forwards incoming client connections in round-robin fashion to the back-end servers. As a result, all back-end servers receive roughly an equal number of client connections.

* **Weighted Round Robin:** With this method, the load balancer forwards incoming client connections to the back-end servers in proportion to the weight assigned to these servers. Each server is assigned a default weight of 50, which may be customized to any value between 0 and 100. 

	As an example, if there are three application servers A, B and C, and their weights are customized to 60, 60 and 30 respectively, then servers A and B will receive an equal number of connections, while server C will receive half that number of connections. 

	**NOTES:** 

	* A weight of ‘0’ against a server means no new connections are forwarded to that server, while existing traffic continues to flow as long as it is active. Using a weight of ‘0’ can gracefully bring down a server and remove it from service rotation. 

		**NOTE:** You may not be able to set a weight value of ‘0’ with the Beta software. 
	* The server weight values are used only with the 'weighted round robin' method. They are ignored with 'round robin' and 'least connections' load balancing methods. 

* **Least Connections:** With this method, the server instance serving the least number of connections at a given time receives the next client connection. 

###Health Checks
The health check definitions are mandatory for each of the back-end application porta. The port number and protocol must match as well, otherwise the configuration is rejected. 

The load balancer conducts periodic health checks to monitor the health of the back-end ports and accordingly forwards client traffic to them. If a given back-end server port is found unhealthy, then no new connections are forwarded to it. 

The load balancer continues to monitor the health of these unhealthy ports, and resumes their use if they are found healthy again by successfully passing two consecutive health check attempts. The health checks against HTTP and TCP ports are conducted as described below. 

* **HTTP:** An `HTTP GET` request against a pre-specified URL is sent to the back-end server port. The server port is marked healthy upon receiving a `200 OK` response. The default `GET URL` is “/” via the GUI and may be customized. 

* **TCP:** The load balancer attempts to open a TCP connection with the back-end server on a specified TCP port. The server port is marked healthy if the connection attempt is successful. The connection is then subsequently closed. 

	**NOTE:** The default health check interval is 5 seconds. The default timeout against a health check request is 2 seconds and the default number of retrial attempts is 2. 
	
These health check parameters are currently not available for customization in the Beta. They will be exposed in the final GA release. 

###Session Persistence
The load balancer supports ‘source IP’ based session persistence against a given VIP port. As an example, if session persistence is enabled for port 80 (HTTP), then subsequent HTTP connection attempts from the same client (same source IP) are persisted to the same back-end server. 

The load balancer supports a maximum of 10000 client persistence entries. The expiration time for these entries is 10 minutes. Additional requests received from the same client after 10 minutes may get forwarded to a different back-end server. If the session persistence entry has not expired, but the back-end port has become unhealthy, then a new server will be selected for forwarding subsequent client connection.  

###Preserving end-client IP address 
The load balancer works as a reverse proxy, which terminates incoming traffic from the client, and establishes a separate connection to the backend serverinstance using its own IP address. For HTTP connections with the backend servers (against front-end HTTP or HTTPS connections), the load balancer preserves the original client IP address by including it inside `X-Forwarded-For HTTP` header. For TCP connections, the original client IP information is not sustained.

##SSL Offload
For HTTPS connections to VIP, the load balancer terminates incoming SSL and forwards plain-text HTTP connections to the back-end server. The back-end servers are thus offloaded from performing CPU-intensive SSL handshakea and encryption/decryption tasks, and can use all their CPU cycles for processing application traffic. An SSL certificate is required for the load balancer to perform SSL offload tasks. One may either use a pre-existing SSL certificate or purchase a new one, and mange it through the [Bluemix/Softlayer Certificate Store ](https://control.softlayer.com/security/sslcerts). 

**NOTE:** TLS version 1.2 is supported with SSL offload, while SSL v3 is not supported due to its vulnerabilities. The SSL cipher list is not customizable at this time. 

##Advanced Traffic Management
This section discusses advanced traffic management using the load balancer.

###Max Connections
Use the ‘max connections’ configuration under the VIP port to limit the maximum number of concurrent connections possible for a given VIP port. By default, no limit is specified. The maximum possible concurrent connections against an individual VIP port or across all VIP ports is 64000. 

###Connection Timeouts
There are three internal system timeout settings. These settings cannot be customized. 

* *** Timeout for Server-side Connection Attempt (in seconds) – 5 seconds:** This is the maximum time window load balancer will use to establish TCP connection with backend server. If the connection attempt is unsuccessful, then the load balancer will try the next available server per the configured load balancing method. 

* *** Client-side Idle Connection Timeout (in seconds) – 50 seconds:** The maximum idle time after which the load balancer will bring down the client-side connection, if the client previously fails to close its connection properly. 

* *** Server-side Idle Connection Timeout (in seconds) – 50 seconds:** The maximum idle time (with back-end protocol configuration of TCP) after which the load balancer will close the server-side connection. With the back-end protocol configuration of HTTP, if the load balancer fails to receive a response to its HTTP request within the idle timeout time, then it will return an error message to the end-client. 

###HTTP Keep Alive
The load balancer supports H'TTP keep alive' as long as it is enabled on both the client and back-end servers. The load balancer will attempt to re-use server-side HTTP connections to increase connection efficiency and reduce latency.

##Sample Configuration Flow
The GUI screens in this section are mock-up screens only. The actual screens may vary. 

1. Begin by logging into your Softlayer/Bluemix account using your regular Softlayer/Bluemix ID. After a successful login, access the URL provided to you for Beta service. Note that the Beta service will not be visible under the production **Network > Load Balancing** menu. 

	**NOTE:** If you are not previously logged into your Softlayer/Bluemix account, then you will be directed to the login page. Please login using your regular Softlayer/Bluemix ID, and then try accessing the Beta access URL again.
	
	Click **Add New** to begin.

	![]()

2. Pick your data center (Beta is available in a few select data centers only), review the plan information and click **Next**. 

	The Bluemix load balancer ‘Beta’ service will carry no charge. At the end of the Beta interval, please manually delete your Beta service instance.

	SCREENSHOT

3. Select the subnet which can be reached by your back-end compute servers (VSI, Bare metal).	Select the private subnet to which the load balancer will connect. This subnet must have network connectivity with backend servers hosting your application. This means that the backend servers can either be on the same subnet as the one you select for the load balancer, and/or the backend servers can be on the subnet(s) that have Layer-3 connectivity with the selected subnet (you may have to enable “VLAN Spanning” in this case). 
	
	Note that the corresponding public subnet is automatically selected by the implementation. The load balancer uses two IP addresses from the private subnet, and three IP addresses from the public subnet. If sufficient IP addresses are not available, then the order of subsequent provisioning will fail.
	
	SCREENSHOT
	
	**NOTE:** In case you see a greyed out option for the load balancer, type `Private`, then ignore any further options as they are not supported and will be removed prior to the GA release.
	
4. Configure the load balancer name and description. Also, specify front-end and back-end protocol and port mappings. You can add up to four ports. You may select HTTP, HTTPS or TCP as your front-end protocol. 

	HTTPS traffic must be terminated. As a result, your back-end protocol options are HTTP (for incoming HTTP and HTTPS) or TCP. The front-end port numbers must be unique. A default health check is automatically associated with each of the back-end port. The default load balancing method used is 'round robin' and stickiness is disabled. Optionally, you can change the load balancing method and enable client session stickiness per port.

	SCREENSHOT

	For HTTPS protocol configurations, you may reference your pre-existing certificate in the certificate store or upload a new certificate.
	
	SCREENSHOT
	
5. Next associate your application servers (compute instances hosting your application) to the load balancer. Ensure that these backend servers have network connectivity with the private subnet chosen earlier in step ‘3’.

	SCREENSHOT

6. Accept the Master Service Agreement and click **Create** to create your service.7. You will be directed to the summary table, which displays the load balancer service instance you just created. Please pay attention to the **Status** field. A status of 'Pending' implies that an operation is in progress in the background. No new configuration changes will be accepted until the operation completes and its status changes to ‘Running’. You may need to refresh your screen to see the latest status.
 
 SCREENSHOT
 
8. Clicking on the service name on this page takes you to the service overview. 

	**NOTE:** The screenshot is an example only. All tabs and information within these tabs may not be visible with the Beta service. The screen contents may change at the time of GA.

	SCREENSHOT
	
9. You can also navigate to **Server instances** and **Protocols** tabs to edit your existing configuration.

	SCREENSHOT
	SCREENSHOT
	
## Beta AppendixDue to limited availability during Beta phase, we suggest you create no more than one Load Balancer Beta instance per account.### Inter-operation with Security Groups (Beta) ServiceThe load balancer Beta service does not interoperate with the Security Groups Beta service. If a Security Group Beta is defined against a virtual server instance (VSI) and this VSI is used behind your load balancer Beta service, then you must take one of the following actions:* Remove the Security Group Beta service defined against VSI.
* Add a rule in the Security Group Beta to allow for all incoming traffic destined towards the port that the application is listening on. This port is identified as a back-end port in the load balancer configuration.### Required Permission for UserThe load balancer Beta service can only be created or managed by users having ‘Manage Load Balancers’ permission. 

SCREENSHOT


## Known Limitations

This section should contain a description of any known limitations of the offering, and any possible workaround procedures to follow. If there are no limitations, this section can be omitted.

## Getting Support

This section should give the information for contacting the support team, in various ways, and it also should let the reader know if there's special information they need to include when contacting the support team, such as a customer ID number, or a system log file.

## Giving Feedback

This section might be used in Beta releases only, but maybe we should always invite customer feedback? It could specify the support team's contact information, or even possibly a way to reach an OM for the offering, with that person's permission.
