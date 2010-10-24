---
layout: post
title: "Appropriate Technology for Appropriate Circumstances"
published: false
---

We've all seen it sometime in our careers: there's either a manager, another
colleague, or even ourselves, who blathers on about a hot new technology
that ought to be implemented or investigated, since everyone else is doing it.

This is bad.  Why?  I'm not exactly the most reserved person when it comes
to hot new software or technologies; however, sometimes we are blinded by
the new technology's potential without even looking at the problem we're
attempting to solve.  I'm guilty of this, and I'm sure many of us did the
same thing early in our careers (note, I'm not saying it's a bad thing to
be totally in love with a piece of software or technology: what *is* a bad
thing, however, is that we're solving a non-existent problem or we've asked
ourselves the wrong question before springing for the new piece of software.)

A great example of this is the steady adoption of NoSQL databases.  These
databases are *great* because they're speedy and it makes it real easy for
someone to scale up the load that can be processed.  However, what some people
don't realize is the nature of the data that they're "storing": sometimes,
it makes absolutely no sense to store things in such a database that
reads/writes into memory, especially when you're only doing a simple drop-in
replacement for an RDBMS.  Why?  Do a quick test: kick the power cord to your
MongoDB server.  Oh, shit.  What happened to your data?  The server forgot
to commit the data to disk right away? Oh shit, oh shit, oh shit.  As long
as your expectations line up with the extreme volatile nature of these
key value stores, then it'll work wonders for you.  Otherwise, you're just
using a hammer on a screw.

Another great example is choosing programming languages for a specific task.
Often, the best programming language (for us) is one that allows us to
express our thoughts quickly.  However, the best programming language,
for the business, is one that fits the problem domain and/or the politics.

