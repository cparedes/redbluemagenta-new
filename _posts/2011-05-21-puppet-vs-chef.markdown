---
title: Puppet vs. Chef
layout: post
published: true
categories:
- Puppet
- Chef
- Configuration Management
- System Administration
- DevOps
date: 2011-05-21 19:25:00
---

I'm approaching this from the view of a systems administrator - it's possible that
my world view is quite different from a developer, so my preferences might seem
a bit weird to a couple of people.  I'm not going to outright say which one is
"better" than the other - in fact, both CM systems are quite awesome, it's just that
they both have different philosophies on how to approach configuration management.

Puppet, for example, explicitly exposes the dependency graphs - if you want to
impose order, you are *required* to declare how you're imposing it.  Is it
because of a dependency?  Is it because you've grouped resources in different
stages and you want your resource to execute in one of those stages?  The DSL
forces the sysadmin to think about exactly how their system is built - ideally,
you'd setup dependencies between various disparate resources, and each group
of resources should be able to execute independently of each other (for example,
I could setup a group of dependencies that describe a web server running Rails,
and another group of dependencies describing NFS services - unless the web server
depends on NFS, I should be able to setup the web service and the NFS service
independently from each other. If one set of resources fails to execute, the other
set should still be able to execute regardless of the state of the first set of
resources.)

Puppet has a Ruby DSL - however, there's a lot of institutional momentum to continue
writing in the old Puppet DSL.  Writing manifests in the old Puppet DSL enables
us to write things more clearly, but at the cost of flexibility - for example,
the way I've been applying an action on each element of an array is by using
"definitions":

<pre><code>
define foo {
  # do something to ${name}
}

...

$bar = [ 'one', 'two', 'three' ]

foo { $bar: }
</code></pre>

In Chef, I'd instead write it like this:

<pre><code>
%w{one two three}.each do |foo|
  # do something to #{foo}
end
</code></pre>

For slightly harder things that rely on data structures such as arrays and hashes,
Puppet makes it harder to clearly express what I want.  But there's a danger
with Chef: you can write arbitrary Ruby code, which can sometimes obfuscate exactly
what you want to do on the system, or at the very least, can make it confusing to
read compared to equivalent Puppet code.

Chef is much more conducive for programmers.  You can declare dependencies like
in Puppet, but by default, the resources execute in order as they're listed in
the Chef recipe.  You can write arbitrary Ruby code alongside Chef DSL code -
however, aside from the dangers outlined above, you'll also have to be very careful
about the execution order of Ruby code and Chef DSL code.  Ruby code always executes
during the compilation stage (unless you wrap the Ruby code within a "ruby_block"
resource declaration.)  The compilation stage executes before the execution stage -
this can lead to things like this:

<pre><code>
# foobar.conf, before cookbook_file resource:
hello

# foobar.conf, as defined in cookbook:
hi

# Recipe:
cookbook_file "/etc/foobar.conf" do
  source "foobar.conf"
  mode 0644
end

grab_state = %x[cat /etc/foobar.conf]
</code></pre>

grab_state will only be "hello" on the first execution of this recipe - otherwise,
it will have the value "hi" in every other execution.  This might be confusing,
even if grab_state is located below "cookbook_file" in the recipe.

This will do the right thing, if you expect "grab_state" to be given a value *after*
"cookbook_file" does its thing:

<pre><code>
cookbook_file ...

ruby_block "grab state" do
  block do
    grab_state = %x[cat /etc/foobar.conf]
  end
end
</code></pre>

This is because we've now placed our arbitrary Ruby code within a Chef DSL resource,
which is executed *after* the compilation phase.

In short, the largest differences between Puppet and Chef is not only the language,
but also the way they approach configuration management.  Chef breaks the mold
by enforcing execution order - most other CM systems execute things
in random order, which is just simply a different way of looking at things
(the description is more important than *how* we get there.)  Puppet, in my
opinion, is neither better nor worse than Chef, but simply different - there
are pains in programming in both.  Other issues, such as administering the server,
are also just "different" - Puppet has quite a bit fewer dependencies, but also
doesn't come with as many things that are taken for granted in Chef (such as
an external node classifier, and other things that make Chef a bit better
in larger deployments, such as a separate data store, queues, etc.)  Chef Server
is a pain in the ass to setup, and is totally overkill for small deployments,
but once you do get it setup, you do get a lot more language features out of
the box than you do in Puppet.

VERDICT: DIFFERENT
