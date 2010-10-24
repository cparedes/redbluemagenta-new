--- 
layout: post
title: "Dilemma: deciding between three major UNIXES as a desktop"
post_id: "217"
categories:
- Computers
- Freebsd
- Linux
- Opensolaris
---
<p>There are a few considerations that I'll lay on the table first, which influence my decision (and motivate the reasons why I'm having a dilemma as well):</p>
<ol>
<li>I want solid RAID support for four equal sized hard drives.  ZFS is the most ideal filesystem for me to use, but it is only really supported in FreeBSD and Solaris.</li>
<li>Desktop responsiveness is key.</li>
<li>I want nVidia 64-bit drivers for 3D acceleration, just in case I decide to play a bunch of games off of Steam.  This somewhat rules out FreeBSD in the meantime, but there is promise for 64-bit drivers in the near future.</li>
</ol>
<p>Now here is my dilemma: after working with OpenBSD and FreeBSD, I generally love the BSD way of doing things; for instance, all of the BSD's are extremely elegant in implementation and typically strive for technical excellence, which shows in how everything is implemented in the base system (for instance, do you want sound to work? kldload snd_foobar.  Want sound to work with multiple applications?  sysctl dev.pcm.0.play.vchans=4.  It is MUCH better to deal with than ALSA and PulseAudio.)  However, the only BSD that would have a chance as a viable desktop for me (FreeBSD) seems to somewhat fall through when it comes to having, say, RAID-Z as a boot drive (though I also hear this is an issue on OpenSolaris, too.)  I can't spare any extra SATA ports for a mirrored zpool for a boot drive, so I might just have to make do with what little I have on that front.  That, and I can't seem to get my X-Fi Forte to work with OSS4 on FreeBSD, either.</p>

<p>A lot of the hardware I have owned and used to own have, well, worked in Linux just fine.  But I do say that with slight reservation, as I had a lot less trouble with OpenSolaris and *BSD when it came to hardware support that <strong>worked well.</strong>  A lot of Linux distributions can also be needlessly complicated as well; while I do admire Debian's massive cleanup of the /etc directory and the massive amounts of example configuration files and comments in each script and configuration file, the niceties can be a bit of a double-edged sword: more often than not, I either have to scroll through a wall of text (when I could've just looked at all of that information in a man page or an FAQ) or work strictly with the built-in scripts (of course you don't HAVE to, but to keep everything nice and tidy within the system that is already setup, it can be a bit of a hassle.)  But hell, the Linux kernel has come a long way, though I believe there is still a terrible bug with large file transfers since 2.6.18: as soon as a large file transfer has started and sustains for a few minutes, the CPU usage spikes all the way up to 100+%.  This is unacceptable for any sort of desktop environment.</p>

<p>I have not had too much experience with OpenSolaris, but the deal breaker in this case is that there are not enough packages to download off of their repositories.  However, a lot of the more exciting things have come from this project, such as dtrace and ZFS.  It'd be great to investigate further at the very least.</p>
