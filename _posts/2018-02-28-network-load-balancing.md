---
title: Network load balancing — Key to secure, reliable communication
date: 2018-02-28 16:00:00 +0530
categories: [Networking Design, Load Balancing]
tags: [OSI Model, Networking, SSL, DDOS]
render_with_liquid: true
image:
  path: /assets/img/posts/2018-02-28/cover.png
---

The Internet went public in the 1980s and gained massive popularity after a decade. In the mid of 1990s, a Single server could not serve the incoming traffic of the most popular websites.

When using several servers to expand the capacity, Those companies needed a way to access those multiple servers using a single domain address. The server farm needed to be hidden from external users. The solution should give a seamless experience without bombarding all the requests to a specific server.

## Initial load balancing systems

![Round Robin based request routing](/assets/img/posts/2018-02-28/round-robin-routing.png)
_Round robin based request routing_

To solve this problem, the Round Robin Domain Name System, the most basic form of Server load balancing mechanism, was invented. The Round Robin domain name system (DNS) based load balancing method has a list of internal IP addresses belonging to multiple servers in the server farm, residing in the demilitarized zone (DMZ) behind a firewall that belongs to a single domain name.

When a user requests a resolution of a website name from the domain name system (DNS), it replies with one of the IP addresses from this IP list. That IP is changed to the next candidate IP for the subsequent request. As the name implies, this candidate IP address rotation is done using a round-robin algorithm.

## Limitations of Round robin based DNS load balancing

Although the scalability issue was successfully addressed with this new domain name resolution approach, which allowed adding an almost limitless number of servers to a single domain name, this method later became inconvenient mainly due to the lack of fault tolerance.

When the server farm grows, there is a high chance of bad servers being unwilling to serve user requests due to reaching the server's processing limits or unseen issues. Since there is no way to know the server's status, Round Robin DNS keeps routing requests to that server which makes the situation worse by offering a bad experience for the end users.

Moreover, record caching and client-side address caching rendered this method almost unusable on certain occasions. This method cannot guarantee Reliability and high availability when the server farm is growing.

Businesses needed a solution to balance the load, not just a load distribution method like Round Robin DNS, But also a proper fault tolerance mechanism considering network congestion, transaction time, and server load when deciding to route the request.

## Load balancing on the modern technology stack

With the advancements in technology, load balancing is applicable for many areas other than server farms, such as computation clusters, availability zones, storage arrays, different network links, databases, etc. Also, load balancing takes care of reliability, increases responsiveness, ensures security, and provides the ability to scale out computational clusters dynamically.

We have hardware and software load balances with modern technologies to perform these tasks. This load balancing operates on two load-balancing operational strategies: server-side or client-side load balancing.

## Hardware and Software load balancing

Load balancing can be performed by dedicated physical network devices or with software components. Hardware-based load-balancing tools are often called Application Delivery Controllers (ADC).

By separating load balancing from an application component to a dedicated hardware component, It is possible to run server health checks at regular intervals. These hardware devices use network layer techniques like Network Address Translation (NAT) to securely route inbound and outbound traffic without exposing internal networks to the public domain.

## Hardware load-balancing vs software-based load-balancing

Although hardware load balancing offers excellent performance with efficient routing mechanisms using integrated circuits explicitly designed to handle network stack, software load balancing is also catching up quickly. In addition to cost benefits, software load balancing provides excellent flexibility and scalability regarding cloud and micro-service-related load balancing.

We have several dominating load balance tools in both hardware and software forms. For example, F5 BigIP, Cisco, Fortinet, and Citrix are leaders in hardware load-balancing solutions. At the same time, NGINX, HAProxy, and ELB provide solid software-based load-balancing solutions. Nowadays, these load-balancing solutions can offer a competitive, rich set of functionalities.

## Layer 4–7 load balancing

![OSI Model](/assets/img/posts/2018-02-28/osi-model.png)
_OSI Model_

There are two main modes for network load balancing with reference to the OSI Model, Layer 4 load balancing or Layer 7 load balancing.

Layer 7 load balancing operates on the highest level of the Open Systems Interconnection (OSI) model. Layer 7 load-balancing products take routing decisions based on application layer information like https/http protocols, headers, message body contents, cookies, etc.

Layer 7 load balancing is more expensive than layer 4, which requires high processing power and decision-making time due to the complexity of the layer 7 network packet information. But layer 7 load balancing provides excellent flexibility and efficiency compared to layer 4.

Layer 4 load balancing products inspect packet data and use network address translation (NAT) to do the routing. Since layer 4 load balancers cannot see the complete request packet and responses, they cannot manage the network traffic based on application logic scenarios. Due to that fact, layer 7 load balancing is more popular than layer 4 load balancing solutions in general application development and cloud clusters.

## Additional features of load balancers

In addition to traffic load balancing, load balancing solutions can provide a rich set of other features since it acts as a facade interface for internal resources scrutinizing incoming/outgoing traffic. Some features are as follows.

**SSL offloading:** With this feature, we can offload the SSL encryption and decryption tasks from web servers to the load-balancing device handling the traffic. Since those operations are resource-hungry, relieving web servers from SSL validation saves the computation power needed to process client requests.

**Content compression:** We can optimize bandwidth usage significantly by compressing the payloads. Offloading inbound and outbound content compression to a load balancer saves the computation power of the web servers and still gets the benefit of bandwidth utilization when delivering the content to the external network.

**DDoS Protection:** Since the load balancer acts as a gateway between internal and external networks, Distributed Denial of Service (DDoS) attack mitigation can be performed in the load balancing layer. Load balancers with intrusion prevention (IPS) and web application firewall (WAF) can effectively detect and prevent application-level layer 7 DDoS attacks.

## Server side load balancing

![Server side load balancing](/assets/img/posts/2018-02-28/server-side-load-balancing.png)
_Server side load balancing_

Server-side load balancers act as a gateway between internal and external networks and bridge external traffic to correct internal nodes based on network load, latency, server health, server responsiveness, etc. The server side is the widely used load-balancing mechanism in on-premise server setups and cloud-based designs. And often use layer 7 load balancing with added features like intrusion prevention, web firewall, and DDoS mitigation.

## Client side load balancing

![Client side load balancing](/assets/img/posts/2018-02-28/client-side-load-balancing.png)
_Client side load balancing_

In micro-service architecture designs, application functionality is broken down into several small services, often named micro-services. Multiple instances of each micro-service are deployed to serve the load to ensure high availability and scalability. Application functionality is a composition of all these micro-services; hence, a single client needs to call multiple micro-services to fulfill a workflow.

To ensure an efficient operation, the load must effectively be distributed to different types of these micro-services. Following the conventional server-side load balancing mechanism for these scenarios will create a monolithic design with layers of load balancers with a single point of failure. When using micro-service architecture, many calls happen between endpoints, even for one transaction, which overloads the conventional server-side load balancer and fails the system altogether.

Moreover, this will limit the scalability factor as well. Having a server-side load balancing for micro-services can be a significant bottleneck for the system.

Having a separate load balancer per micro-service type can be a viable solution. But, it complicated the design and acted like a monolithic behavior since different loads originating from other microservices must route via those central load balancers.

Also, those load balancers must distribute high velocity of small but complex requests to micro-service instances accordingly. The Overall design complicates adding and removing new instances from micro-services clusters, hindering automatic scaling-up operations. This design will introduce arrays of load balancers when communicating between services. When component numbers grow, this will be a real bottleneck.

A client-side load balancer for each microservice type is the accepted approach for micro-service architecture. Each client load balancer is responsible for routing requests to available destination micro-services.

Also, one load balancer only handles all the traffic generated by that specific microservice type. Client-side load balancing eliminates the single point of failure, simplifies service management, and provides the capability to scale up or down seamlessly based on traffic. This approach allows the auto-provisioning of instances and elastic scaling to be executed smoothly while supporting enhanced fault tolerance mechanisms.
