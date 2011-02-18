---
title: Parallelizing common tasks
layout: post
published:false
categories:
---

I've finally gotten around to trying out [GNU Parallel] yesterday, and it's got me
thinking about common sysadmin tasks that can be parallelized easily with GNU Parallel
or other pieces of software.

Of course, we'll have to think about the algorithms/tasks that we're trying to execute.
Things like establishing SSH connections in a for loop are typically "embarassingly
parallel", since the return result of a particular SSH connection isn't necessarily
used in any subsequent SSH connections.

Anything that requires the use of a TTY are typically NOT parallelizable, though
you can typically work around this by either finding out how to pass in variables to
the program without invoking the interactive interface, or by using `expect` (though
it looks like from [this thread] that `expect` isn't thread-safe, although it seems
like it's still possible to have `expect` handle multiple processes at the same time.)

Other tasks, such as transforming a set of files with `sed`, merging all of the results
into a single output, and then piping it into a file is mostly parallelizable - you'd
have to process all of the files, then wait until all of the results are available
before running `cat` against all of the different pieces (you probably don't want to
have each thread write to a file as soon as it's done; there's too much margin for
error.)

[GNU Parallel]: http://savannah.gnu.org/projects/parallel
[this thread]: http://groups.google.com/group/comp.lang.tcl/msg/80db2ba9a5371f18