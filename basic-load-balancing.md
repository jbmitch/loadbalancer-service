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

# Basic Load Balancing
The IBM Bluemix load balancer service distributes traffic among multiple server instances (bare metal and virtual server) that reside locally, within the same data center. 

## Public Load Balancer 
Each IBM Bluemix load balancer service is assigned one IPv4 Virtual IP (VIP). This VIP has a globally-unique IP address, which is accessible over the public Internet. The backend compute servers (hosting your application) must be on an IBM Cloud private network. 

**NOTES:** 

* The VIP is not associated with any domain name. You may use an external domain service to map your custom, user-friendly domain name to this load balancer VIP address.

* As a good practice, we recommended that you provision your backend servers as ‘private-only’, unless they require direct public connectivity. This practice helps achieve better security, and it preserves your public IP address. The applications hosted on these backend servers are still accessible over public network via load balancer VIP. 

### Front-end and Back-end Application Ports/Protocols
You may define up to four front-end application ports (protocols) under VIP and map them to respective ports (protocols) on the back-end application servers. The VIP and the front-end VIP ports are exposed to the external world. The incoming user requests are received on these ports. 

On the other hand, the back-end ports are only known internally. These back-end ports may or may not be the same as the front-end ports. As an example, the load balancer may be configured to receive incoming web/HTTP traffic on front-end port 80, while the back-end servers are listening on custom port 81. 

The supported front-end ports/protocols are HTTP, HTTPS and TCP. The supported back-end ports/protocols are HTTP and TCP. Incoming HTTPS traffic must be terminated at the load balancer to allow for plain-text HTTP communication with the backend server. 

**NOTES:**

* During the initial configuration, you can define up to two front-end ports only. Once a load balancer is created, you can edit port configuration to define additional ports, up to the maximum of four ports.

* All four VIP ports must map to the same set of back-end server instances.

* The maximum number of server instances that may be placed behind a load balancer is 1000.

* The port range of 56500 to 56520 is reserved for management purposes and cannot be used under VIP.

## Load Balancing Methods
The following three load balancing methods are available for distributing traffic among back-end application servers:

* **Round Robin:** Round Robin is the default load balancing method. With this method, the load balancer forwards incoming client connections in round-robin fashion to the back-end servers. As a result, all back-end servers receive roughly an equal number of client connections.

* **Weighted Round Robin:** With this method, the load balancer forwards incoming client connections to the back-end servers in proportion to the weight assigned to these servers. Each server is assigned a default weight of 50, which may be customized to any value between 0 and 100. 

	As an example, if there are three application servers A, B and C, and their weights are customized to 60, 60 and 30 respectively, then servers A and B will receive an equal number of connections, while server C will receive half that number of connections. 

	**NOTES:** 

	* A weight of ‘0’ against a server means no new connections are forwarded to that server, while existing traffic continues to flow as long as it is active. Using a weight of ‘0’ can gracefully bring down a server and remove it from service rotation. 

	* The server weight values are used only with the 'weighted round robin' method. They are ignored with 'round robin' and 'least connections' load balancing methods. 

* **Least Connections:** With this method, the server instance serving the least number of connections at a given time receives the next client connection. 

## Health Checks
The health check definitions are mandatory for each of the back-end application ports. The port number and protocol must match as well, otherwise the configuration is rejected. 

The load balancer conducts periodic health checks to monitor the health of the back-end ports and forwards client traffic to them accordingly. If a given back-end server port is found unhealthy, no new connections are forwarded to it. 

The load balancer continues to monitor the health of these unhealthy ports, and resumes their use if they are found healthy again by successfully passing two consecutive health check attempts. The health checks against HTTP and TCP ports are conducted as follows:

* **HTTP:** An `HTTP GET` request against a pre-specified URL is sent to the back-end server port. The server port is marked healthy upon receiving a `200 OK` response. The default `GET URL` is “/” via the GUI, and it can be customized. 

* **TCP:** The Load Balancer attempts to open a TCP connection with the back-end server on a specified TCP port. The server port is marked healthy if the connection attempt is successful, and the connection is then closed. 

	**NOTE:** The default health check interval is 5 seconds, the default timeout against a health check request is 2 seconds, and the default number of retrial attempts is 2. 

## Session Persistence
The load balancer supports session persistence against a given VIP port based upon the `source IP` of the connection. As an example, if session persistence is enabled for port 80 (HTTP), then subsequent HTTP connection attempts from the same client (same source IP) are persistent on the same back-end server. 

The load balancer supports a maximum of 10,000 client persistence entries. The expiration time for these entries is 10 minutes. Additional requests received from the same client after 10 minutes may be forwarded to a different back-end server. If the session persistence entry has not expired, but the back-end port has become unhealthy, a new server is selected for forwarding any subsequent client connections.  

## Preserving end-client IP address 
The load balancer works as a reverse proxy, which terminates incoming traffic from the client. It establishes a separate connection to the back-end server instance, using its own IP address. For HTTP connections with the backend servers (against front-end HTTP or HTTPS connections), the load balancer preserves the original client IP address by including it inside the `X-Forwarded-For HTTP` header. For TCP connections, the original client IP information is not preserved.
