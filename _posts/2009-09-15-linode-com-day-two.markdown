--- 
layout: post
title: "Linode.com: Day Two"
post_id: "212"
categories:
- Arch Linux
- Computers
- Debian
- Linode
- Linux
- Vps
- Xen
---
<p>I've been with <a href="http://www.linode.com/?r=8bc45a7ce694f24356ff5e8e62177650598f9ab1">Linode</a> for about two days now.  I can't say anything but good things about them when it comes to an unmanaged Linux box.  They provide a VPS (Virtual Private Server) which they call a "Linode," and each Linode is sitting on top of a Xen hypervisor.  I've subscribed to the Linode 720 plan, which grants 720 MB of guaranteed memory to the VPS, 32 GB of HD space, and about 400 GB of transfer per month.</p>

<p>I have heard about the pros and cons of a VPS versus, say, a shared web hosting account, and a dedicated server, namely that a VPS often has to compete for resources and it's often simply used if you want root level access to a box without necessarily expecting the greatest amount of available resources.  However, my Linode has been churning away quite happily on a fairly unoptimized installation of Apache 2 and MySQL 5, which are the usual memory eaters in a server.  In fact, I've heard of stories of people running <em>game servers</em> on even the lowest priced Linodes, which is typically unheard of in any other VPS solution; any other VPS, from hearsay at least, is usually oversubscribed and often competes for computer resources, especially for disk i/o access.  Linode.com seems to have either tweaked their Xen hypervisor installs to be quite generous in keeping the servers responsive, or they do not oversubscribe (or both!)  I'll have to experiment with running a Half-Life 2 server to see how much I can stretch the resources in my VPS.</p>

<p>I've gotten my hands dirty with both shared hosting and an unmanaged server; a shared web hosting account is definitely cheap and it depends on which provider you go for, but it's ultimately insufficient if you want to have root level access to a box, and also if you want to have 99.99% uptime for your website.  What's great about shared hosting, however, is that they often host the sites on pretty decent hardware, even if it's being oversubscribed.  A dedicated server is quite costly, usually consists of subpar outdated hardware, and will, more often than not, be underutilized.</p>

<p>A VPS seems to be the best of both worlds: there's a lot of customers on one computer, yet offers the capabilities of a dedicated server.  Thus, more often than not, the VPS is hosted on a box with, say, twenty or thirty other customers, with the box usually featuring multiple CPU's and a robust array of hard drives.  On my VPS hosted by Linode.com, they use a quad core Xeon L5520 clocked at 2.26 GHz and a RAID-10 array of hard drives.  Further, each Linode/VPS on each box has access to all four CPU's for multithreading.</p>

<p>As for Linode.com's unique features, there's a nice web frontend for deploying distributions on the hard drive (which by the way, if you wanted to wipe your entire installation and wanted to deploy a new distribution, only about five minutes) and for managing DNS records.  Also, they provide out of band login to your machine via SSH without having direct network access to your box.  This has saved my ass many times; I've ended up toasting an Arch Linux installation and was able to recover the installation via their out of band SSH interface.  They also provide graphs of network traffic, CPU usage, and disk i/o.  Speaking of disk i/o:</p>

<pre class="brush: bash">$ sudo hdparm -Tt /dev/xvda
/dev/xvda:
 Timing cached reads:   9414 MB in  1.99 seconds = 4721.44 MB/sec
 Timing buffered disk reads:  382 MB in  3.00 seconds = 127.33 MB/sec
</pre>

<p>Not bad.  I haven't done too much other testing yet, other than briefly running a 24 slot TF2 server.  However, I took it down after calculating how much bandwidth would be required each month in order to run such a server; I think it would be much safer to run a single 4 or 8 player L4D server.</p>

<p>In any case, I highly recommend Linode.com to anyone who wants to host a website or run a light game server (or any other server application) without breaking the bank.  I've personally subscribed to the Linode 720 plan, which is PLENTY for me to work with.  I actually host this website and another domain with the same Linode; I'm potentially going to open a Left 4 Dead server on this box as well.</p>

<p>Don't forget to click on <a href="http://www.linode.com/?r=8bc45a7ce694f24356ff5e8e62177650598f9ab1">my referral link</a>!</p>
