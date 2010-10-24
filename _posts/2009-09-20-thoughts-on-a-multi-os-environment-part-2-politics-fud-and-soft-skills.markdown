--- 
layout: post
title: "Thoughts on a multi-OS environment part 2: politics, FUD, and soft skills"
post_id: "213"
categories:
- Computers
- Fud
- Linux
- Management
- Politics
- Soft Skills
- Windows
---
<p>As a help desk technician, I sometimes forget about some of the issues when it comes to managing staff, budgets, and other things of that sort.  Usually I strive for technically excellent solutions which make sense for the end user, and when such a thing does not exist, I try to at least get the end user to be as productive as possible at the lowest level possible (does the computer turn on, can I type and save my reports, etc.)</p>

<p>However, as part of my job function, I also take an interest in the health of the infrastructure, as sometimes the lower level issues may stem from a problem in the infrastructure itself.  For instance, imagine the end user attempting to access the internet.  She sees a "page not found" error.  Uh oh, maybe she hasn't pulled an IP address from the DHCP server, or maybe the DNS server is down; then I'll be pulled away from the problem and see the same exact thing from Joe's computer.  So now I am lead to think that either the DHCP or DNS server has gone down, or that the wiring in a particular section of the building is faulty.</p>

<p>To make it easier on everyone, I want to strike off as many scenarios from my mental list as possible.  Sometimes, this requires setting up an additional server which acts either as a failover server or as a primary server for a particular service (obviously, I can't do this while the two people from the scenario above are flailing their arms around trying to get their work done, so I'd have to of course come up with a stop gap solution for both of them.  Let's assume, from now on, that I've already implemented such a solution for those two people.)  Occasionally, the best solution is one that employs either FreeBSD or Linux servers to provide the service (and yes, there is a use for Windows servers, I do not want to spread FUD about Windows servers); there is an extremely solid reputation for BSD/Linux servers when it comes to providing low level services, like DHCP, DNS, SMTP, etc.  However, this is often not enough for IT managers and other high level administrators, as there may be a bit of FUD surrounding the use of Linux as a server.  Here are some of the things that they may say:</p>

<ul>
<li>Linux is free, therefore it has no value for the enterprise.</li>
<li>There are no vendors to be accountable for our systems when they take a nose dive.</li>
<li>Linux has no security measures.</li>
<li>Linux is old.</li>
<li>Linux is written by a bunch of amateurs with no eye for business planning.</li>
</ul>

<p>In order to convince administrators to allow for Linux servers to be placed on the network, we must be diplomats; we cannot simply talk about FOSS philosophy and denigrate the Windows platform as a closed system.  We also cannot talk about the technical merits of employing Linux as a tie-in with Windows servers, as higher level administrators often are worried about business decisions and training costs.</p>

<p>So how do we work around this?  Again, we work as diplomats, always understanding of the business impact: talk about how there is a lower TCO for Linux and do some research in order to confirm that this is indeed the case (Microsoft provided some questionable research in the TCO of Linux vs. Windows; even if this were true, I would be weary about it mostly because the research was not done independent of the company.)  Talk about how the average salary for a Linux sysadmin is a few more bucks per year, yet there is a much lower initial buy-in for software than Windows.  Talk about how it is not simply just a bunch of amateurs writing code for all of the applications and the kernel for free, but rather, there are many programmers employed by IBM, Red Hat, etc. who contribute to the entire ecosystem (and also, be prepared to talk about how Linux was based off of UNIX, which was around since the 60's.)  Talk about how Google, Amazon, IBM, etc. have based their entire infrastructure off of Linux and BSD, and how ecosystems and organizations like Wall Street, the French Police, etc. have been running on Linux and have saved money and time by running a free operating system.</p>

<p>But also, this requires some knowledge of the Windows ecosystem as well.  Do your research, setup Windows servers, and don't be totally ignorant as well.  I, personally, am ready to concede that Exchange and Active Directory are extremely good pieces of software that have helped organize email and the organization at large much better than anything I've seen on Linux so far.  But I know enough to know that it is absolute suicide to use Windows for anything that is mission critical.</p>

<p>In short, put yourself in the shoes of the IT manager and other administrators handling the budgets and try to think about what they are worried about.  They are worried about vendor support, training costs, etc. Then do your research and present an alternative as diplomatically as possible.</p>
