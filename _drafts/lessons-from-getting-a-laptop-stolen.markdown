---
title: Lessons from getting a laptop stolen
layout: post
published:false
categories:
- Systems Administration
- Thief
---

I've recently had my laptop stolen last weekend while I was out drinking with
a couple of friends at a bar in Seattle's University District.  Along with
my laptop was an iPad that was also in the same bag.

Up to this point, I did not care much about physical/computer security -
foolishly enough, as a system administrator, I should have cared much more
than I did.  At the very least, this is an embarassing incident; really, I
should've known better to leave my bag unattended for even a minute.  At worst,
it was a very expensive mistake.

There's a number of lessons that I've been reflecting on since last Saturday,
much of which has to do with computer security practices as well as physical
security.

1. **Keep your bag in sight at all times.** This is a bit zealous and is likely
impossible (especially when you're with a bunch of friends and you need the
table and chair space for other things), but I figure that if you can try
to have your bag at least next to you, or on top of a table, that you'd likely
remind yourself much more easily that you have to grab the bag before going
somewhere else.

2. **Encrypt essential data partitions.** I didn't do this. It's unlikely that
the thieves will try to extract the data from the hard drive, but just in case,
make sure that if you're using Mac OS X, that you use FileVault (and maybe for
Linux, you can use TrueCrypt.)  The only recourse, if the would-be thieves
decide to scrape the data from the machine, is to simply throw away the data,
if they don't know the password to unlock the partition.  It's probably good
practice to have a throw away username/password setup on the machine with
fake data just in case the thieves decide to use rubberhose cryptanalysis. :)

3. **Use random passwords for everything.** Just in case they guess any of
your passwords, you'll at least compartmentalize the damage if you have
different random passwords for everything.