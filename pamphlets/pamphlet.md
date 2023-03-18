## lesson1

## lesson2
![](../img/2-1.png)

## DNS
What a system mostly comprises of?

### Browser/client
The facebook app is also a client.

This doesn't scale well:
![](../img/2-2.png)

This does(what happens when you make a req for a website):
![](../img/2-3.png)

## 4- Web Servers
### The communication between browser/client and DNS

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

## 5-Load Balancer (Part 1)
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

## 6-Load Balancer (Part 2)
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

## 7-Databases

## 8-Caching
## 9-Jobs - Servers
## 10-Jobs - Queues
## 11-Services (Part 1)
## 12-Services (Part 2)
## 13-Data
## 14-Cloud Storage CDN
## 15-System Design Reminder
## 16- Principles of System Design - Availability
## 17- Principles of System Design - Reliability
## 18-Networking - OSI & TCP/IP
## 19-TCP IP
## 20-TCP Explained
## 21-UDP