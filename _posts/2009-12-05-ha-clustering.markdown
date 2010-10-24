--- 
layout: post
title: HA Clustering
post_id: "218"
categories:
- Computers
- High Availability Clusters
- Networks
- Systems Administration
---
First, I want to forewarn everyone that I do not have any experience in deploying HA clusters whatsoever, though I do have some experience in deploying HPC clusters; this is simply a thought experiment in discovering any sort of caveats and issues with HA clusters and what sort of things should be addressed in either policy or implementation.

The objective in deploying HA clusters is to help provide either full or N+1 redundancy for a service, where "service," in this case, means any sort of cohesive set of servers, applications, and monitoring systems that fulfills a promise to the customer (if the customer, for instance, wants a website servicing 100,000 unique hits per day that has 99.9% uptime, a web service could include some sort of reverse proxy in order to reduce the load on all servers, multiple servers that are either copies of each other and/or divided up depending on what task they are assigned, a firewall, and network monitoring systems in order to make sure that everything runs fine.)  Where HA clusters fit in this puzzle is where we need servers that are (close to) exact copies of each other.

In a small cluster, the interconnects between servers are often ethernet cables; this offers bandwidth up to 1000 Mbps, or around 500-700 Mbps throughput.  This is fine if there are either only a lot of changes to small files, or occasional changes to large files.  If, however, we are anticipating many changes to large files, then we will either have to delegate any sort of file storage to a SAN, or implement faster interconnects between the servers (using InfiniBand or maybe splitting the network load between several ethernet cables.)  However, this may not be an issue if we do not require all of the servers to have a consistent state with the main server; a scenario where this may be the case is where the main server handles all of the load until the moment when it goes down, which then it goes to another backup server to pick up the slack.

Another way that we can address state consistency is to only send the least amount of data that keeps a specific file / database consistent; that is, if say a text file contained a list of names on the main server, and another name was appended to the end of the file, we only send the difference between the original file and the subsequent change to the file, which will lessen interconnect bandwidth requirements but could also increase processing costs for the server computing the differences between files, depending on the complexity of figuring out the differences between files (it could be a database which requires calling a function that spits out the difference between the original state and the modified state.)

Anyway, this is mostly just for my information; these issues could all have been addressed in the past by either crafty SA's or vendors, it's just a matter of reading more about the state of the art.
