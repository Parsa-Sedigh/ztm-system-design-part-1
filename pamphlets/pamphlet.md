## lesson1

## 3

## DNS
What a system mostly comprises of?

### Browser/client
The facebook app is also a client.

This doesn't scale well:
![](../img/2-2.png)

This does(what happens when you make a req for a website):
![](../img/2-3.png)

## 4
Domain name system

IP address is sth computer recognizes as a unique address somewhere on the web.

Many companies that own websites, especially big ones like facebook, they have hundreds, thousands of IP addresses and the reason is
they're numerous potential servers in the world that can house all the different files to build facebook on our browser.

Therefore a website doesn't necessarily need 1 specific IP address, but they can have many IP addresses but each of these IP addresses
are unique.

So facebook has millions of IP addresses that all pretty much aim to do 1 thing: give the end user the appropriate files to see facebook.com .

IPV4 has `4.29 X 10^9` amount of bits. IPV6 has `3.4 X 10^38`. DNS has to maintain both of them.

A website has at least 1 IP address.

The request itself, it's often going to have the specific format of IP that it wants back(IPV4 or IPV6). Now it might be the browser or
might be the request sender. So we ask DNS: give me the IPV4 of facebook.com .

DNS can figure out based on your locations and other factors, which IPV4 or IPV6(if multiple) to give to you.

## 5- Web Servers
### The communication between browser/client and DNS
![](../img/3-1.png)

web server == application server

It's a server somewhere, takes a req and sends back the appropriate files to the client that it then uses to either build the whole website
or build a portion of it. Why a portion? Well, maybe the client sends a req for some json data instead of files.

The core things that make a web server work:
- application logic: code we write. This lives on the webserver
- database: Is outside of webserver

The more traffic(reqs to webserver), the more resources(storage, memory(RAM), processing power(CPU), network speed) the webserver needs.

How do we increase the capacity of a web server?

There are different types of scaling:
- vertical: we need more resources, so we increase the resources on **one** server
- horizontal: clone the webserver virtually (multiple webservers)

What is virtual cloning?

We're copying webserver nodes. Virtual means they're duplicates of application logic.

We'll talk about this.

## 6-Load Balancer (Part 1)
When cloning the webserver, the application logic is also gonna get cloned because the logic runs independent of the server. It doesn't care
what server it's running on.

However does it make sense to duplicate the data in DB?

No.

![](../img/6-1.png)

How do we horizontally scale those webservers in order to service all those users?

We need to use load balancer.

Some assumptions

what is latency?

Latency is the time it takes for the browser to fire req and get back a res from server. That total travel time from when the browser
starts making the req to when it receives a response, is latency.

### Assumptions
When we horizontally scale the webservers, there are some problems to be solved.

For better understanding the example, let's make some assumptions:
- all reqs have the same latency
- all servers have identical resources

Without a load balancer, we have no way to figure out accurately how to manage the req coming in. Maybe all of the reqs are going to one server.
We need to manage the load(number of reqs being made).

Load balancer sole purpose is to evenly distribute the load between available servers.

Load balancer acts as the entrypoint into the app space(system).

A strategy to how distribute the load is called **round robin**. So how the load balancer chooses what server to route a req to, is
a strategy and round robin is a common one. This strategy says: we're gonna go in order of the list of servers we have access to.

In real life, this might not be the most accurate strategy, but you're **in theory**, getting a very balanced load, because of our 2 assumptions
above.

Note: The load balancer is smart enough to know that if a server goes down, it's not gonna route any req to it, instead it's gonna distribute the load
with the remaining servers that are up, so this is another thing that load balancer can do.

Load balancer is the traffic cop.

## 7-Load Balancer (Part 2)
Load balancers manage the amount of traffic that each webserver receives from the numerous number of browser/clients.

### Session persistence
You're constantly making reqs to the webserver in order to receive new data that is specific to the session you have open.
So you have a persistent session where there's info that the server needs to know about what you're doing, so that it can give you up to date
information relative to your session. This means the communication between your client and server has to be consistent. Meaning you can't
jump between multiple servers and one client. The reason for this is because server is the one that is aware of what it has given the session so far.
This means you need to maintain some type of persistent connection between a client and a webserver.

Load balancer keeps track of that session persistence. Meaning the session between user1 and serverA is maintained and load balancer knows whenever
it receives a req form user 1 about that resource, it's gonna forward it to serverA again because that's the established connection.

So when we need session persistence, load balancer can do this.

When a server goes down that has retained sessions, load balancer knows that it needs to re-distribute the connections to that down server, to other 
servers.

Q: How other servers that are gonna get the sessions of the down server, know the session information?

There are different strategies of how you retain that session information. You can keep that info on the client and then the new servers that take
over, they'll have to sped some time computing to make sure that they catch up to what server A was doing.

This computing is gonna happen regardless. There needs to be some additional time that new servers that take over, need, to catch up with the sessions.
This was regardless. But where you store that information, is different. You can store it on clients or load balancer(2 strategies).

So if we have session persistence, if a server goes down and other servers need to take the session, there will be additional computation time,
in order for the servers to catch up to what's happening. So we'll have some delay.

![](../img/7-1.png)

## 8-Databases
DB purpose -> store and retrieve data

The idea of storing and retrieving from a DB, typically ends up being the most latency-heavy steps in our system.

Let's say we wanna show a post to all people of north america, we don't want the req of each of them to go through the db operations which are
time consuming. We introduce a caching service for this.

## 9-Caching
### Database vs caching service
Both are two forms of storage.

Caching service prioritizes speed of retrieval over amount of data stored. But DB stores a lot more.

Since caching service stores much less data, you're able to retrieve data out of it much faster.

So when we choose to use a caching service, you're doing so with the intention of storing high-impact data that you need to pull out fast.
So this is where sth like region-specific data that is accessed significantly, is stored.

For the first time, we get the data from DB but when we want to send the response back, at the same time, we will send those data to caching
service to cache it(after doing some transformations from the data sent by DB).

In subsequent reqs, we use the cached data.

## 10-Jobs - Servers
What is a job?

What is business logic that is handled in webservers?

It's the code that delivers a functionality specific to the business. Any code you write that has to eventually interface directly with the user,
is business logic. Everything else, we call `application logic`.

### Business logic vs application logic
Application logic is still driven by the code that you write and it looks similar to business logic, but the context is not necessarily 
around the user anymore.

Application logic runs on job servers.

The news feed on facebook is coming from DB. How do we make sure that our DB has all of the **up to date** posts that a user might query for? Because
we're storing those posts from news-outlets, magazines, blogs which are called data sources.

In order for us to get these up to date data into our DB from data sources, we need some step to do that which is where job servers step in and in
job servers, we have application logic.

Storing up-to-date posts should happen in background and independent of the users interactions and we can put this into application logic which run
in job servers.

job = 1 unit of work

unit depends on the context.

job is some computational work that we need to do in order for our app to have the appropriate functionality or data that it needs for users.

Everything that is not user-driven, is application logic.

How does the job server know it's time to start running these jobs?

This is where a job queue steps in.

Job servers perform on jobs.

## 11-Jobs - Queues
Job queues are a way for us to understand how the job servers know when to perform these jobs work.

In our example, we define a job as our job server reaching out to any single of those data sources, fetching it and then storing it in the DB.

Sth has to **send** the jobs into the job queue. This is where we come back to web servers. Yes, we said web servers are driven by user behavior.
You can see sometimes that webservers might not always be driven by user behavior, they might also house some application logic not just 
business logic and webserver can be the thing that queues and figures out when to start setting uo these jobs. A cron jobs are an example for this.
For example we queue our jobs in a specific point in time because that's when we know we have the latest posts for that day. This is an 
example of cron job. This cron job can be set up on webserver.

So webserver can send these jobs into the job queue.

## 12-Services (Part 1)
Webservers communicate with services. Services encapsulate all the components we design in systems but as a service.

In facebook, we have 4 main features that make facebook like authentication, messenger, newsfeed and communities. We're familiar with
using them all inside of one system, but we can isolate each of these into their own independent system or **service**.

## 13-Services (Part 2)
Newsfeed service in nutshell:
![](../img/13-1.png)

A service is a complete project, it's isolated in a sense that it focuses on one single responsibility(microservice architecture).
In other words, a service is a self-contained system that delivers a very specific functionality.
![](../img/13-2.png)
Note: Load balancer is the entry point for each system and service.

## 14-Data
### Data hose and data warehouse
User behaviors and interactions are sent from the webservers to data hose and data hose will pass it to dat warehouse.

We can do this through the webserver, or we can do this directly from the browser/client. So depending on where you host your
data hose, you may be able to expose it publicly, so the browser/client can capture user behavior directly in the app, it doesn't have to 
always be fired from the webserver, you can also fire it from the frontend. For example running the google analytics on the frontend.

Regardless from wherever you get the data into data hose, the data hose will take the raw data and message it into a format that we wanna
store in data warehouse, but we still keep the raw data and the reason is we're gonna send both to the data warehouse.

Data warehouse is a storage solution.

So data hose is the entrypoint for all of the raw data of user actions, behaviors or events that are firing in your system and then
data hose will modify and change the data and we store **both** raw and changed data in data warehouse.
![](../img/14-1.png)

## 15-Cloud Storage CDN
Cloud storage is a backup source of data and it lives in the cloud so that it doesn't live on your local machine. So other than DBs, you might
wanna use additional storage.

We can also have a line between webservers and cloud storage so that we can store not just the data coming from the data hose but also
any data in your whole system. So essentially anywhere that we can store data on our system, we can back it up on cloud storage.

we can store these on cloud storage(like amazon s3):
- app files(the files we pass to browser to build the website)
- media files (like amazon s3)
- raw data(that go into warehouse)
- formatted data(goes to warehouse)

We can backup **any** kind of file or data into cloud storage.

### CDN(content delivery network)
The purpose of this, is to make it more performant to deliver our content of website to many users around the world.

CDN is gonna hold the files you wanna serve to browser, so that the browser won't go to webserver.

DNS will work with CDN to figure out the nearest edge server based on the user's location, it gives the IP address of the nearest edge server instead of 
origin server. This gonna work for users all around the world.

This reduces the latency.
![](../img/15-1.png)

## 16-System Design Reminder

## 17- Principles of System Design - Availability
- availability
- reliability

### Availability
Availability is the percentage of time within some given period that your system or app can be used.
![](../img/17-1.png)
![](../img/17-2.png)

### Percentages of uptime we wanna aim for
![](../img/17-3.png)
![](../img/17-4.png)

Cloud providers aim for 2 to 4 nines.

## 18- Principles of System Design - Reliability
Reliability talks about whether or not within your system it crashes or creates some failure when it's being accessed.

Mean time = average time
![](../img/18-1.png)

The result of the image above is 60 hours. Meaning that on average(mean), within a 300hr time period, there are 5 failures which means every
60 hours there's some kind of critical failure.

With reliability calculation, you're trying to determine within the actual available time period of system(previous lesson), how much failures
are there going to be?

We're trying to figure out whether or not the system is reliable to perform the function without error. You want to reduce the amount of errors or
you wanna increase the meantime between the failures.

Reliability is not as important as availability because we can get around these errors by also try to make system more available.

## 19-Networking - OSI & TCP/IP
OSI = open system interconnection

General model for network communication between 2 systems. This model breaks communication into 7 abstract layers.

As long as there are 2 computers that are communicating, you can use the OSI model to abstract out how the communication is happening.
![](../img/19-1.png)

Each of these layers has their own protocols.

Protocols are a set of rules that define how this data should be structured and then communicated. They're gonna be structured in terms
of the syntax(how it's written), the semantics(what key terms there are that are holding some kind of meaning), synchronization of how
this communication is possible such as how much data gets transferred at a time and error recovery and ... . These are all rules that
define the communication of data at a specific layer.

---

application layer:

---

presentation layer:

presentation layer = translation layer

In this layer, the data we get from the application layer, is extracted and then it gets manipulated and formatted so that it's ready
to be translated across the network. What does that transformation look like? Might be encryption or decryption or compression or 
translation.

---

session layer: Is the layer that is responsible for opening and closing the communication between two devices. As we know, sessions
open and close and there's a certain period of time in which that connection is alive. So the session layer is responsible for making sure
that a session stays long enough, in order for all the data to be properly transferred and then closed so that it's not wasting resources,
when it's done.

---

transport layer: responsible for end-to-end communication between two devices. It makes sure the actual communication happens.
This includes taking data from session layer and then splitting it into chunks and segments before sending it to the next layer underneath.

---

network layer: responsible for communication between major groups of networks(inter and intra networks). So if there was 2 separate
LANs and they had to communicate with each other, the network layer is handling that. In other words, this is for communication between
**separate** networks.

---

datalink layer: facilitates communication between 2 devices on the **same** network.

---

physical layer: has to do with transferring bits of the data into a physical cable.

## 20-TCP IP
### Transmission control protocol / internet protocol
TCP/IP: How computers connect to the internet and transmit data between them.

TCP has builtin error checking. It's designed to ensure that data does actually get transferred completely.

IP tells us the address of where we're trying to go.

TCP has to do with ensuring that everything actually is getting sent to the final destination.

The TCP/IP framework is 4 layers(the right one is TCP/IP layers):
![](../img/20-1.png)

- application layer: dictates the protocol that the application needs
- presentation layer: converts the data coming across, into the format that the application requires and vice-versa
- session layer: maintains the communication session open, so that all data gets properly transferred

## 21-TCP Explained
In TCP, both ends can receive and send data to each other(2 way connection - data can go both way). Now in order for this to happen,
we need to make sure that we're on the same page. We need to ensure that we're getting the correct order of data and the data is all received.
This happens with establishment of **ISN**(initial sequence number).

Both ends have their own ISN and it's a randomly generated number. It's a way for us to track on the server and client whether or not we have
received the correct ISN from the opposing system. So if the client wants to send data to server, it's gonna send it's ISN and server will
respond: Hey, I did get your number, I'm ready for whatever data is associated with this number. Because as the numbers change,
it knows what the corresponding data for that number will be. So we're using ISN to track what the data is going to be and because both
sides are trying to send data to each other, they both need to keep track of each other's ISN for that connection.

The ISN represents the data being sent over for that current session connection that is established. A different connection is 
different data all together and that's why ISN is also randomly generated.

Example for TCP:

The client wants to establish a connection to the server, so it will randomly generate an ISN. It will then send a synchronization req
to the server with that ISN. The server then is gonna acknowledge that it got the client's ISN. The way it acknowledge that it received
that synchronization req with that number, is it's gonna add 1 to that number. After receiving the incremented value, the client says:  
Ok, I probably synchronized with the server. But remember the server also wants to send data back to the client. Meaning it it self also
needs to synchronize back with the client with it's own ISN. So the server will generate it's own ISN as well and then it will also
send as part of that acknowledgment, back to the client also a synchronize message as well, saying: Hey, my ISN is Y.

The client is now done 2 things:
- it's recognized that it's been acknowledged of this value X + 1
- it's also gonna acknowledge that the server has sent it's ISN of Y 

So client is gonna send back an acknowledgment with ISN of Y + 1

So server in the acknowledgment that it sent, said: I acknowledge I'm ready for X + 1 and our client said: I acknowledge I'm ready for Y + 1.

The reason for this is now when we send data between two, we're gonna send it with data including the ISN of X + 1 and the server is gonna
send data to the client with ISN of Y + 1.

The moment we received data and we see inside that this ISN has the actual acknowledge ISN incremented, we know what session, what data
is correlated to, in our communication.

For example:

Our client is saying: Hey, for this current session that I'm sending you data, I want you to know the data is associated to 123(ISN).
Then server is gonna say: I acknowledge that and for the session, when you send me data, send it to me with the ISN of 124, so that I know to
correlate what you sent me to 123.

The reason for this ISN + 1 is because data gets sent in chunks.

Since we're receiving numerous data potentially, maybe over multiple TCP connections, how do we ensure that we know that which connection is 
associated to which data?

We associate them with ISN. Each connection has it's own ISN.

HTTP and HTTPS are examples of TCP.

![](../img/21-1.png)

Why? Because we need all the parts of website for it to function correctly, so we use TCP with it's guarantee delivery of data.

In TCP we need to transmit data between 2 directions(data flows in 2 directions, both can send and receive data). In TCP, there are 3 
handshakes which establishes the connection.

## 22-UDP
### UDP/IP vs TCP/IP
UDP = user datagram protocol

UDP is faster, simple and efficient than TCP. Because there's no connection. There are less resources consumed in order to maintain that 
connection between client and server. There are no sessions in UDP.

In UDP, the client sends a req to server. That req allows server to know there's a client out there that wants some data. Now the server is gonna
keep bombarding the client with data. Whether or not that data gets over there, the server doesn't care. It doesn't need to guarantee that
data delivery.

Majority of time, you're gonna use TCP unless you're trying to build live streaming or VOIP.
![](../img/22-1.png)

## 23-Proxies
- forward proxy: sits between client and internet. So it's not in our system, it's in the client side. Now, the client will never directly
access the internet, they'll have to access it through the proxy. Then, the internet sends these reqs to our servers in our system.
The forward proxy can hide the details about the clients from internet and servers. Forward proxy can modify the reqs. It can even lock out
access to websites.
![](../img/23-1.png)

- reverse proxy: It lives on our side(inside of our system). The internet will send client reqs(or forward proxy reqs) to our reverse proxy before
it hits our servers. Since the reverse proxy is the entry point into our system now(from any reqs of the clients), the internet and therefore
then the clients will see when they get responses, is that anything comes from the reverse proxy. So we can hide any details about where our
servers are, maybe we can modify some of the data on proxy and ... and there are other advantages of a reverse proxy for us other than hiding
the details.
![](../img/24-1.png)

Reverse proxy: This proxy kinda behaves like a load balancer. Because it's receiving all the reqs as an entry point and then it's forwarding
these reqs to the servers. So we can use a reverse proxy as a load balancer. In fact many load balancers are reverse proxies. An example of
proxy is nginx, many system use nginx as a load balancer as well as a reverse proxy. So that is one of the benefits of using a reverse proxy
as a load balancer which is you get the other benefits that come with it(when using a reverse proxy as load balancer, you get the benefits of
a load balancer but you also get the benefits of reverse proxy), such as: 
- caching: You can cache a lot of what you need to deliver to
clients on the proxy. The caching service we talked about can be the reverse proxy as well. So when we get req, we just send a res from 
reverse proxy and this is way faster than having to forward the req from reverse proxy to servers and then servers doing their work and ... .
- security benefits
- another benefit is load balance requires having multiple servers, because load balancer needs to manage the load across multiple servers.
But you can get the benefits of a proxy even if there's just a singular server. Because you still get access to caching and obfuscation of
our servers from clients.
![](../img/24-2.png)

Generally in many systems, we're gonna use a reverse proxy as a load balancer. So when you think of a load balancer, you can also think
about using a reverse proxy in that place.

From now, when we talk about load balancer, we're talking about using a reverse proxy in that place.

## 24-Exercise: Imposter Syndrome

## 25-Load Balancing Strategies
New component: Reverse proxy / load balancer

- round robin: reqs gets evenly passed over to every server in the list. Everytime we get a req, we send it to the next server in the list.
The issue with this strategy is servers are often independent of one another and they don't have any knowledge of the other servers. As a result,
it's very easy to have some type of issues within the structure of doing round-robin because the servers oftentimes have different amount of
resources, bandwidth and ...(also the reqs are different than one another). Now even if the reqs are the exact same and consume the same amount
of resources, the servers themselves might have a different amount of resources available to them.
- weighted round robin: Each server has a weight(depending on how much resource they have). The higher the number, the more resources.
Another advantage is we can easily AB test things. Because let's say server1 has the new changes that we wanna make and we wanna a majority of 
traffic, to server 1 so we can test things. So we can AB test based on the data of performance. So this was another benefit of breaking up
the traffic un-evenly based on server resources or server needs.
![](../img/25-1.png)
- Least connections: This has to do with either sessions or depending on how many reqs are on the server, we determine that we pass req to the
one that has the least. In img, between server 1, 2 and 3, 3 has the least, so we're gonna pass the req to server 3. Now if multiple servers has
the same number of req, we can rely on round robin again or even weighted round robin.
![](../img/25-2.png)

Any component inside of the system can overload, go down or other failures.

Note: If Reverse proxy / load balancer goes down, the entire system breaks. What we call this is **single point of failure**. When you have
a single component that can go down and destroy the functionality of your entire service or at least a core functionality of your service.
In this case, we want to redundant the load balancer. If the primary load balancer crashes, we can swap it with backup load balancer.
![](../img/25-3.png)

So redundancy allows us to reduce single point of failure. Maintaining the backup load balancer increases the cost and complexity.

## 26-Server Clustering
A technique you apply to a component where you increase the amount of that component(like a server) that exist.

With Reverse proxy / load balancer, the servers act independent of one another. Each server has no idea about any other server in the system, 
it doesn't care, they don't even know about each other.

Clustering doesn't replace load balancing.

Let's take server 1 as an example for explaining clustering:

Clustering is where you have a bunch of servers that act together and are completely aware of each other and work together as a cohesive single
unit towards some objective.

For example server 1 was composed of multiple other servers that are all working in conjunction. Each of these servers working together is known
as node.

In the image below, since they are nodes, you wanna see these 5 servers together as 1 server. They all respond to 1 IP address, they share 1 
IP address, they're treated as a single entity. As a result, you need to assign a leader node. Let's say we chose node 1 to be the leader.
So when the req comes in, they all go through node 1. Then, node1 acts like a load balancer / reverse proxy and it will forward
these reqs evenly between the remaining nodes. But the different between load balancers and clustering is that in load balancers outside of this
cluster example, none of the servers are aware of each other. However, inside of a cluster, all the nodes are not only aware of each other,
they're constantly communicating with each other and they're constantly syncing their states. By syncing their states, if any one node
goes down, that's very easy for the remaining nodes to pick up the work that that one which is down now, was doing. There's no latency between this
and that's the whole idea of clustering: You're trying to reduce node-to-node communication but you also making it easy for any of the nodes
to replicate the state of a node that went down. All of these nodes hold the exact same resources. They don't share resources per se, but they have
the identical resources, they're identical servers or nodes in this case.
![](../img/26-1.png)

Passive nodes and active nodes: When we have single point of failure, we introduce one or more passive nodes or servers or components in general.
In the img, we have 4 nodes that are all active and sharing the load, but this might not be necessary, you may be over-utilizing
resources to keep 4 nodes active, if the traffic is not that much. What we can do is maintain passive nodes, so we can make nodes 4 and 5 passive.
They're there, but they're still syncing with nodes 2 and 3, there's still communication between these, the passive nodes are just not routed
the traffic.
![](../img/26-2.png)

Now if nodes 3 or 2 go down, then nodes 4 and 5 will become active. This is whole idea of keeping these passive nodes
![](../img/26-3.png)

When having clustering, the overall server 1, 2 and 3 are not aware of each other, **inside** of these servers, they are their own cluster of
node servers which are aware of each other. This is load balancing and clustering.

Servers 1,2 and 3 aren't aware of each other:
![](../img/26-4.png)

## 27-Databases Intro
## 28-CAP Theorem CP
## 29-CAP Theorem AP
## 30-ACID and BASE Properties for Database Selection
## 31-What's Next?
## 32-Thank You!