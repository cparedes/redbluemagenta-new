---
title: More thoughts on DevOps
layout: post
published: true
date: 
categories:
- System Administration
- DevOps
date: 2011-03-03 19:04:00
---

After working at a few places, watching interviews, and thinking about system
administration culture, I've ended up slightly changing my views about the DevOps
movement.

Automation, repeatability, consistent root cause analysis: these are all things
that any decent sysadmin strives for.  We are all problem solvers, and we all
either want to have good documentation, automation, or both!  This new culture
that's developing is a great thing - it's of my opinion that automation is key
for system administration, and that requires a bit of coding, as well as knowing
systems in and out.

I was talking to a friend of mine who works at Amazon, and he mentioned that everyone
in the company is, indeed, a systems engineer in some capacity.  This is what
DevOps strives for - software engineers who have systems clue, or at the very least,
two groups who collaborate very closely with each other.

HOWEVER: Amazon, Google, etc. have had to adopt this strategy because they were forced
to do so at the scale that they were operating at.  [Benjamin Black had a great
interview on NoSQL Tapes](http://nosqltapes.com/video/benjamin-black-on-nosql-cloud-computing-and-fast_ip) which covers roughly the same ground, though mostly on the
technology rather than the culture (to summarize what he said, you want to look at
your application and infrastructure, figure out what the problem might be, and then
fix it; similarly, before adopting new technology, see whether the semantics of the
new technology actually fits in with your infrastructure and application.)

Similarly, DevOps tackles an issue that ought to be tackled, yet it's operating
at the "wrong scale": we do need to automate stuff, but some sites might be fine
with just writing shell scripts, some sites might be fine with a few Puppet/Chef
recipes but might not to manage their entire site that way.  Some companies end
up assigning SA's to program stuff in applications, or vice versa - sometimes the
result is disastrous! (Why?  I _know_ I can't write clean code worth crap \[sorry
folks who have to maintain my code\], and I know of a few instances of
devs attempting to manage systems with Chef and neglecting certain operational
issues, such as performance tuning, packaging, or whatever else.)

If we want to execute DevOps well, we should look at Amazon and Google for guidance,
but we should _also_ look at how they're organized, what sort of scale they're
operating at, and see if it actually makes sense to assign roles in the same way.

I'd love to hear your guys' thoughts on this, too.  Comment below. :)
