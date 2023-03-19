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
## 20-TCP IP
## 21-TCP Explained
## 22-UDP
## 23-Proxies
## 24-Load Balancing Strategies
## 25-Server Clustering
## 26-Databases Intro
## 27-CAP Theorem CP
## 28-CAP Theorem AP
## 29-ACID and BASE Properties for Database Selection
## 30-What's Next?
## 31-Thank You!