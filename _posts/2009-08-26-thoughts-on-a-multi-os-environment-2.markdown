--- 
layout: post
title: Thoughts on a multi-OS environment
post_id: "257"
categories:
- Active Directory
- Computers
- Freebsd
- Linux
- Networking
- Windows
---
I have always wanted to experiment a little more with multiple OS integration in a computer network.  It seems that Active Directory is what's driving most networks these days, and while there might be opposition against a multi-OS environment on any side of the debate, it's a hell of a lot better to factor other operating systems into the equation.  But why would one want to do that, if say most of the users were used to either Linux/BSD or Windows?

It's mostly because it allows for a lot more flexibility.  I'm thinking that if there was an appropriate infrastructure that could support different types of operating systems (and perhaps the right culture to allow for it as well), it would be much easier to come up with fixes for certain problems.  Of course it won't all be elegant as most admins would probably love to have it.  For instance, say there is an AD network at a corporate office, and say they host a website on their own servers.  You could, of course, keep it all Windows-based and make everything easy to manage from AD.  However, as we should all know, IIS is riddled with security holes, and so it's absolute suicide to go Windows-based all the way, <em>depending on what services you run</em>.  That is to say, if you don't host a web server, then it's probably a-okay to simply run everything on a Windows server.

But what about low-level services that provides essential information for client computers, like DHCP, DNS, etc.?  From what little I've read, it's also quite unwise to run these sorts of services from a Windows server; most of these things, at least in ISPs, are run from a *BSD box (or sometimes even a Linux box).  But if you are running an AD network, you don't have to throw the entire baby out with the bath water if you choose to run these services on a *nix box.  I don't know exactly how one would implement such a setup, but from a few quick glances at Google results, it's definitely possible to do.  What I would love to do is to setup a primary DNS server on AD (so that all of the niceties of administrating an AD setup is still there) and then setup secondary and tertiary DNS servers on two FreeBSD boxes.  I've heard success stories, but I'm still a little leery about the implementation details and whether it would break other services, like Exchange or whatever.

In any case, a multi-OS environment would at least balance out some of the strengths and weaknesses of each (server) OS on a network.  Active Directory seems to be an <em>excellent</em> way of organizing a network into a hierarchy which mirrors the actual meatspace organization in a company.  However, for anything that has to do with interaction with the internets - <em>especially</em> low-level services - it is a hell of a lot better to go with *nix machines.
