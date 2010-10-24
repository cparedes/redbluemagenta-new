--- 
layout: post
title: Lessons from administering the Cafe Allegro network
post_id: "707"
categories:
- Café Allegro
- Systems Administration
---
I've administered and lead a small group of volunteers to administer Cafe Allegro's network.  It consisted of an OpenBSD router, a managed HP ProCurve switch, and four access points.

We failed.

Earlier today, all of us agreed to drop the project and to move on.  We've had a maximum of three people on our team at all times, and as soon as it got down to two people, we've become more and more aware that our schedules were getting packed tighter as time went on.  In order to relieve the pressure of having to juggle school/work and running over to Allegro to respond to an incident, I've disabled the OpenBSD box and hooked the cable from the switch directly to the modem.

I want to publicly thank Gabe Storm, Phillip Garland, and Jeff Johnson for all of your guys' help and awesome work that you guys have each put into the network.  I also want to thank Michael Fox and Hoang Ngo for a ton of physical labor that you guys have put in when we were first wiring up the network.

There's a number of lessons learned from this ultimate failure of being able to keep the network up.

<ol>
<li>Do not use a regular old UNIX box as a router, unless there's a platform you can build on top of it that makes it easy to manage.  Most of our pain had to do with the fact that we were using a full blown machine; this gave the router the appearance of being mysterious and being beyond many people's comprehension.  The baristas had an expectation with the previous network configuration that they could just reset the box at will; the OpenBSD box disrupted their expectations.  Corollary: use an appliance or something appliance-like if at all possible.  2nd corollary: Make things able to survive reboots easily.  3rd corollary: LISTEN to the damn customers and primary consumers of the system.</li>
<li>Use monitoring and metrics.  We've learned this the hard way: we'd get a call from one of our close friends who were also regulars at Cafe Allegro saying that the network was down.  Not good.  It's much better to fix the issue before it becomes a customer-facing impact, and to know that the issue exists before the customer does, you must have eyes on the system at all times.  Therefore, setup Nagios. :)  As for metrics, it would've been nice to see trends in bandwidth usage sliced up per hour, just to see when people hop on the network and what sort of loads on the network we could expect.</li>
<li>Have a testing environment ready.  We pushed EVERYTHING to production, including critical network changes.  We <em>did</em> have a git repository stowed on the server, but what good is a git repository on that server when we can't access the server after an erroneous configuration change?  Even if we had a git repository elsewhere, we wouldn't be able to roll back changes if we've inadvertently cut the router from the network.</li>
<li>Hire more volunteers.  Even if assuming we had a perfect solution for dealing with issues remotely, or had a perfect solution with configuration management and whatever else, we would still need admins to rush in and physically fix stuff.  This is unavoidable, especially since we only had <em>one box</em> to deal with, and thus, we had a single point of failure.  We couldn't conceivably raise the money to get failover machines or any of the like, and even then, where would we have put them?  The area where the router was was already cramped as it is.</li>
</ol>

<em>(Note: I credit Phillip Garland for coming up with a couple of the items on the list.)</em>

I've learned quite a bit from this experience, and I'm honored to have served as the resident sysadmin for one of my most favorite coffee shops in Seattle.  We could've done a way better job in maintaining the system, but we went in this knowing that we were a bit green; we knew we were going to stumble at one point as our schedules became even more strained, and it was extremely visible over the past few days (and even sporadically over the past year for various reasons.)

I think the most important item is simply listening to the primary consumers of the product.  I've erroneously assumed that only the coffee shop customers were the consumers; the baristas are the consumers as well, as they have to respond to customer demands as well when people come in and mention that the "internet is down."  We made it way too fucking complicated when it didn't have to be, even if we had lots of control over how we can shape traffic and roll out various services.

Lesson learned.
