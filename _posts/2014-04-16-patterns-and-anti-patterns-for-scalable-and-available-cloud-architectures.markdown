---
title: 'Patterns and Anti-Patterns for Scalable and Available Cloud Architectures'
layout: default
---

More than anything else, architectural choices matter when designing a system with high scalability and availability. Using Azure customers as an example, Microsoft talks about the patterns and anti-patterns they see with their Azure customers and how it affects the four facets of system architecture:


Scalability: Can I add capacity to handle increased demand?
Availability: Will my application endure transient and enduring faults?
Manageability: Do I have ways to understand the health and performance of the live system?
Feasibility: Can I build and maintain the system with my time and cost budget?
Scalability


Scalability comes from two aspects: capacity and density. Capacity involves adding extra hardware, which may be trivial (adding extra web servers behind a load balancer) or very difficult (adding a second database server). Density refers to how efficient you can use the capacity you already have. Traditional performance tuning can significantly increase density.


#Sidebar: Lighting Money on Fire


A common theme during the presentation is “lighting money on fire”. By this Mark Simms means doing something that is inefficient for no reason. Examples include using NATs instead of proper load balancers or XML as an internal data exchange format.


#Metered Resources


Metered resources are something that need to be carefully monitored. An example of a metered resource is a database connection. As a limited resource, misusing it can dramatically reduce density.


To make this more concrete, consider Azure SQL. The standard version only allows 180 connections per database. The default connection pool in ADO.NET is 100. So if you have two web servers connected to an Azure SQL database, and those web servers leak connections, you can easily exceed the cap.


Other examples of metered resources include authentication servers and third party web services. These are sometimes called “hidden resources” because developers often overlook them when planning their architecture.


#Load Leveling through Queues


Spikes in uploads can be problematic, especially on systems that are tuned read-heavy workloads. One way to mitigate this is by trading latency for availability through the use of queues.


Under this scheme new data isn’t synchronously stored in the database. Instead they are placed in a queue which a background process monitors. This background process can smooth out the workload so that the database is used consistently instead of being hammered at some times and idle at others.


Another advantage of using queues is that work can be batched. Generally speaking it is much faster to load information into a database in batches rather than one record at a time.


Finally, this adds a decoupling point. The background process or database could be down entirely without affecting the front-end application’s ability to accept new data.


#Improving Message Queue Availability


If too many messages are received at one time, secondary message queues can be used to store the excess. In order to do this you need to design the application to support multiple queues, even if you intend to only deploy it with one queue to start.


When messages that exceed the size that the application is designed to handle, one technique to avoid data loss is to write the message to blob storage. The logical message in the queue is then modified to store a pointer to the blob entry instead of the original payload.


#Web Server Availability


In order to keep the web server available, all downstream calls must be asynchronous and bounded. The bounds must be in terms of both time outs and concurrent requests. The latter is often overlooked. A somewhat embarrassing example of this is Visual Studio Online’s two hour outage. The root cause of this outage was too many concurrent requests to an external authentication server that had temporarily gone down.


#Authentication Services


This leads us to our next topic, authentication services. When an authentication server goes down it can completely take down an otherwise stable application. For this reason Microsoft highly recommends the use of a federated authentication server.


#Bad Data Logging


Most developers understand the need to validate data, but they don’t know what to do when that validation fails. Merely discarding the data and throwing an error isn’t enough. The bad data should be logged in its raw form so that developers can figure out why the bad request was made.


Most bad requests come from version mismatches. This happens when the user has an older, or newer, version of the client than the server is capable of handling.


#Anti-Pattern: Configuration


When Microsoft’s Azure team review client code they still see hard-coded connection strings and other configuration data. This can be a real problem with the configuration needs to be changed in a hurry to point to different hardware.


#Anti-Pattern: Assumed Database Reliability


For the last generation of developers, database connectivity was a given. Database and internal network failures almost never happen. So it common for developers to not check for exceptions. Or if they do, they don’t handle them correctly and data is lost.


#Anti-Pattern: SQL Injection


Yes, it is still very common problem. In some cases the very first web request they examine has an obvious SQL Injection vulnerability.


#Anti-Pattern: Logging to the Failed Resource


The logging infrastructure needs to be isolated from the rest of the application stack. If logs are written to the same database as the production data, losing one database necessarily means the other is lost as well.


#Anti-Pattern: Rethrowing Exceptions


There are two common anti-patterns in this area. The first is rethrowing exceptions with “throw ex;” instead of “throw;”, causing the stack trace to be lost. The second is rethrowing the exception when there isn’t a higher level handler to catch it. This of course causes the whole application to crash in .NET 2.0 and later.


To see the whole video check out [Building Big: Lessons Learned from Azure Customers](http://channel9.msdn.com/Events/Build/2014/3-633) on Channel 9.