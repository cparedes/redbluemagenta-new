--- 
layout: post
title: Fixing USB Boot for Arch Linux
post_id: "216"
categories:
- Arch Linux
- Computers
- Fix
- Usb
---
<p>So, I was trying to install Arch Linux with a 2009.08 ISO image that was imaged onto a USB stick with UNetBootin.  After trying to boot it up on the laptop I was trying to install Arch Linux on, it kept kicking me into the ramfs$ prompt after timing out on mounting /dev/archiso (the exact error message before it kicked me into the shell was "Waiting 30 seconds for the device /dev/archiso".)  However, it isn't automatically created; you have to create a soft link to /dev/archiso from the block device that your USB stick is assigned to.  For instance, I saw that my USB stick was /dev/sdb (simply from going into /dev, and then typing 'echo *' [since there is no ls command in ramfs for whatever reason] and seeing that /dev/sdb had only one partition).  And then I created a soft link between /dev/sdb1 and /dev/archiso:</p>

<pre class="brush: bash">ramfs$ ln -sf /dev/sdb1 /dev/archiso
ramfs$ exit</pre>

After logging out of the shell, the installer was able to find /dev/archiso (obviously) and continue on with the boot routines.  Hopefully this helps anyone else out there who is experiencing the same issues.
