---
layout: post
title: UNIX and Literacy
categories:
- UNIX
- Literacy
- Programming
- Systems Administration
date: 2010-10-22 19:56:00
---

I was reading through [this article] on UNIX as literature a while ago;
it resonated with me very strongly, if only because I studied philosophy
(and math, which required a lot of prose writing as well) and decided to
hop into system administration as my career.

I started thinking about how UNIX has affected my thoughts on computing and
how I express my thoughts on paper (or, as is the case these days, on the
internet.)  Perhaps it was the "UNIX way" of using pipes between programs
and transmitting clear, parsable text that helped me understand computing on
a different level than, say, doing things the "Windows way"; with Windows, as
a regular user or even as a sysadmin, your main method of expression is through
the mouse: you point and grunt at things, and that's about the extent of your
grammar when communicating with your computer.  On the other hand, with UNIX,
your grammar is expanded quite a bit; you operate on entire sentences, phrases,
and words, and the way you tell your machine to do anything is by writing
things in the command line.

Yes, you might have the same experience in DOS as you might in UNIX; after
all, you do have pipes in DOS and that gets you most of the way there. UNIX
applications, however, often do one thing really really well, and it takes
a bit of knowledge and tinkering in order to figure out how to really use
each program to its full potential.  sed and awk are probably two of the most
powerful programs in the whole UNIX suite of programs, but most people don't
realize it until they start digging deeper.  Hell, I still use sed as a simple
string replacement program, without considering any of the other features
that's available in sed.

But the biggest thing that distinguishes UNIX from all other operating systems
is that *words* and *sentences* are of the primary focus in the OS; it's only
later in the history of UNIX that you start seeing windowing systems, but
even then, text is at the forefront of all interactions in the system.

Powershell, though a valiant effort (and a *better* shell than bash, from
what I hear), is ultimately disappointing.  Why?  The things you operate on
in Powershell are *objects*, not words, and definitely not sentences. This may
be a bit better if we're talking about our perception of the world, for we
at least observe objects before we can ever talk about them with language
(this is a separate topic for a different field, this can be debated for eons),
but we wield words as easily as we move our arms around, and so it is much
more natural to operate on words on a computer than it is to operate on
objects that are constructed artificially.

As the article asserts, it's freeing to be able to use language to express
your thoughts to a computer; I feel the same exact way, too.  It's hard as hell
to get used to it, especially since a lot of us have grown up using Windows,
but once you've mastered how to express your thoughts on a UNIX system, it's
extremely hard to go back to any other paradigm.

All I have to say is:

<pre><code>$ echo "woot" | cowsay
    _____
   < woot >
    -----
         \   ^__^
          \  (oo)\_______
             (__)\       )\/\
                 ||----w |
                 ||     ||
</code></pre>

[this article]: http://theody.net/elements.html
