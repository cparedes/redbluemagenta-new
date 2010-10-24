--- 
layout: post
title: Life as a "junior sysadmin," part II
post_id: "181"
categories:
- Systems Administration
- Work
---
(<a href="http://www.redbluemagenta.com/2009/12/08/life-as-a-junior-sysadmin-part-i/">Here's the link to Part I.</a>)

It's been roughly a year since I've been out of school and have worked as a junior sysadmin for roughly that long as well; in terms of sysadmin-esque experience, I've worked at a research lab, at a high school, and now at a managed web hosting company, and have learned a lot within this short span of time.  My current position has especially taught me a lot in how to do things in a large site that has to run 24x7; for instance, there has to be a lot of caution on which servers to deal with, as one could be either the main load balancer, or it could just be a load balanced slave machine.  And if things must be done on a host that's a single point of failure, then there ALWAYS ALWAYS must be a back out plan.

As an example, I had to check log files that were spread out among 50 different web servers.  In small sites (< 20 servers), we could just do it manually and it'd be fine.  However, with 50 web servers, each generating gigabytes worth of log files spread out among hundreds of log files per server, things have to be automated with a simple shell script.  So I wrote out a script that did a grep on each server for each log file, but while testing the script on a small subset of servers, I ended up killing the I/O on the entire subset of machines.  Not good.  So I had to make sure that if I did have to kill the I/O on each server, it'd have to only hit one server at a time, so that only one server is essentially "out of rotation" at any given time; this is much better than taking down the entire set of web servers, which would significantly impact our web service (thankfully those 50 servers are a subset of all of the web servers, but it'd still take a significant chunk out of our web service.)

The one thing so far that I know I have to brush up on is computer science and programming; I didn't have a complete computer science education in college (I did two years of CS in Seattle University but was totally sick of the software engineering bent that they took), so I'm completely ignorant of a ton of things, like basic data structures other than a hash map and a binary tree, and I also am pretty ignorant about the common algorithms usually used when solving CS problems.  I'm currently learning both Ruby and Perl (I've programmed a bit in Perl for the high school gig, but not too much), and I should probably learn Python while I'm at it as well.  So far, most of the things I've written were in shell scripts, which, for most of the tasks I needed to solve and automate, are probably sufficient enough.  However, I'm likely going to run against a problem which requires much more power than what I can come up with in a shell script, and so I figure I have to be a little more proactive about it.

Thankfully I haven't been put in the on-call rotation yet, so I've no idea what would be involved (though I have had to babysit some stuff in previous jobs late at night, though it wasn't anything like getting called at 2 AM in order to fix a downed server.)

My last few jobs had let me build a few things for the infrastructure, though nowadays, I'm mostly in maintenance mode; I should probably draft up a few pet projects to do for my job, and see if they'll get approved.
