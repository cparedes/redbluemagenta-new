--- 
layout: post
title: A possible universal packaging system for *nix operating systems
post_id: "66"
categories:
- Bsd
- Configuration Management
- Linux
- Package Management
- Pkgsrc
- Ports
- Systems Administration
---
<em>(Note: I don't have any experience with deploying such a setup, but I do have some experience with working in an environment where several *nix operating systems were being used, with a lot of legacy OS's living alongside newer versions; this is a discussion on a possible packaging format that can make the job of deploying packages a little bit easier for each architecture and for each version of an operating system.)</em>

When administering a computing system, it's very possible for each server to diverge in its prescribed system state very rapidly.  We want to maintain some semblance of consistency in the system state so that we know that the service we are providing to the customer is delivered reliably and consistently (for instance, say we want to provide a web service with X response time and Y uptime; we'd have to make sure our system as a whole is serving the correct pages in a timely manner, and we also want to make sure that our system stays up as much as possible.)  One step to this goal is to provide the physical hardware and allocate them to a specific role; another step to this goal is to make sure that the servers delegated to the same role are close to the prescribed system state as possible.

In order to help with the second step, we can deploy a centralized package repository.  However, as time goes on, even if we had a homogeneous system with the same hardware and the same operating system, it's possible for our system to incorporate other pieces of hardware and operating systems that are not equivalent to our original set of servers.  We want consistent functionality, so we still need to be able to provision these new servers with an appropriate set of packages that provide the same functionality.  Say we are using RHEL 5.x servers originally, and we're now provisioning servers with FreeBSD 8.x.  We obviously cannot cleanly use RPM's on FreeBSD (we might be fine if we used the FreeBSD Linux compatible libraries that come with the distribution), nor can we use packages from FreeBSD on Linux (Linux does not come with a FreeBSD compatibility layer.)

We could, however, use <a href="http://www.pkgsrc.org/">pkgsrc</a>.  The packages that it generates might not be universal for every operating system, but there's a singular pkgsrc tree that compiles the same set of source code.  I'd imagine that one could provision a system for each OS being deployed (or setup a cross-compile environment for different operating systems on a single system, sort of similar to <a href="http://tinderbox.marcuscom.com/">Tinderbox</a> for FreeBSD.)  Theoretically, one simply has to set the same package options for each pkgsrc compiled program, and then compile the program and its dependencies for each OS that's deployed (of course, you'd have to make sure that you do a "make package" instead of a straight "make install".)  Then you could NFS mount the package repository on each server, then do a pkg_add /path/to/packages.

But what about post-install configuration?  You'd probably have to either fork the pkgsrc tree locally and design new packages that are based on whatever package is already in the original pkgsrc tree, or edit a few files in the pkgsrc tree itself (which can be bad, since you'd probably want to update the pkgsrc tree from CVS, which could wipe out your changes.)

In any case, I haven't found any decent universal binary package manager that works across all Linux distros and other UNIX operating systems, pkgsrc seems to be the closest thing to such a thing (even if you have to compile everything from scratch.)
