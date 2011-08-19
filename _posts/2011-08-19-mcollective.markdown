---
title: mcollective
layout: post
published: true
categories:
- mcollective
- DevOps
- Systems Administration
date: 2011-08-19 11:41:00
---

Once we're hitting the magic spot of roughly 50 - 100 machines or more, we
usually need to start looking at ways on how to manage all of these machines a
bit more easily.

Configuration management is for keeping servers in spec with what you've
written down.  You deploy packages, configuration, and start services with
configuration management.  For the most part, we've gotten to the point where
we can describe enough in our configuration management language so that we
don't have to login directly to our machines to change anything.

But if say we want to grab information, execute an action, or test something
out on several machines at once, we likely have to look at solutions that can
log into several machines at once and execute an action.  We have applications
such as ClusterSSH and dsh for controlling several SSH sessions at the same
time, general helper applications such as GNU Parallel, and good ol' SSH in a
for loop.  However, we'll also likely have to maintain lists of different
classes of machines depending on what OS it is (what if we wanted to run
"apt-get install foo" on all of our Linux machines, yet we have a few CentOS
machines?), as well as lists of machines belonging to different roles (we
likely don't want to install "apache2" on our DB servers, for instance.)

Further, we might not be interested in stdout output of the commands we run,
nor even the stderr output - we just want to know whether the command succeeded
or failed, and maybe some useful information to go along with it (aside from
attempting to grep out a few things from stdout/stderr.)

mcollective helps with all of these issues.  It's much more than "SSH in a for
loop" - it uses a STOMP broker such as ActiveMQ or RabbitMQ (with the STOMP
plugin) as a pubsub broker, and allows you to use metadata straight from the
server as filters (so you can classify servers in different groups as needed,
and since the server itself is reporting its metadata, information is always up
to date.)  One example of using mcollective is to collect inventory information
and compile a report:

<em>inventory.mc:</em>
<pre><code>
inventory do
    format "%s:\t\t\t%s"

    fields { [ identity, facts["lsbdistdescription"] ] }
end
</code></pre>

<em>Command:</em>
<pre><code>
mco inventory --script=inventory.mc
</code></pre>

<em>Sample output:</em>
<pre><code>
foobar1.example.com:	CentOS release 5.5 (Final)
foobar2.example.com:	Ubuntu 11.04
</code></pre>

Another example is to kick off a Puppet run on all machines with the
"country=us" fact, with no more than three concurrent runs at a time:

<pre><code>
mc-puppetd runonce -W country=us 3
</code></pre>

[Three Drunken SysAds](http://www.threedrunkensysadsonthe.net) also has a nice
article on [using capistrano and
mcollective](http://www.threedrunkensysadsonthe.net/2011/05/deploy-and-roll-back-system-configs-with-capistrano-mcollective-and-puppet/)
for updating code on the Puppet master, and kicking off a subsequent Puppet run
on all of the machines in the fleet with mcollective.

All we had to do in order to query all of these nodes intelligently is not by
maintaining a list of machines on the client, but by directly querying all of
the servers in the fleet, with any server satisfying the filter replying back
to the client.  The servers themselves *are* the source of truth, not anything
else.

--

There's a few caveats that I've run into that should be taken into
consideration:

Security
--------

By default, security is through a preshared key that's distributed between the
clients and all of the servers.  This is bad, especially if you plan on having
mcollective do anything useful, such as run privileged commands or kicking off
Puppet runs.  We've decided to use the [AES + RSA security
plugin](http://docs.puppetlabs.com/mcollective/reference/plugins/security_aes.html)
for encrypting traffic between clients and servers, which is much more secure
than using a PSK.

We've enabled RPC Auditing as well, just to make sure that nothing's amiss.
It'd be nice to centralize all of the logs in one place: we haven't found a
good way to do this yet (mcollective does have a logstash plugin, though it
stuffs all of the audit logs in the STOMP server, which, at this point in time,
can't be pulled from logstash due to a bug in jruby.)  For now, it might be
worth it to install the centralized log audit plugin, pull an aggregate log
file from a single machine, and run logstash against that particular file.

Further, you'll likely want to implement an authorization plugin.  We use the
[ActionPolicy
plugin](http://code.google.com/p/mcollective-plugins/wiki/ActionPolicy) for
authorizing specific users to be able to do certain actions.  For ActionPolicy
to work, you have to make sure your client's computer has UID's synced with the
servers you are querying, since the RPC client grabs the UID from your client
machine, and ships it over as "request.caller" to the server (UPDATE: this is
not true for the SSL and AES + RSA security plugins, only for the PSK security
plugin.)  Since mcollective basically gives you root level access to each
system it runs on (even if it only exposes a few hooks to the client), you'll
want to make sure that you only give just enough permissions to mcollective
that a person (or system user) might need.

Keep in mind that if you're using AES + RSA security or SSL security with
ActionPolicy, you'll want to use "cert=<certname>" instead of "uid=<uid>" - the
AES + RSA security plugin sends the cert name in the "caller" section of the
RPC request, which you can confirm if you enable RPC auditing and drill into
the logs.

STOMP
-----

This was a little annoying, since we already had a rabbitmq server up and
running for logstash and I didn't want to spin up another machine just for
mcollective.  Thankfully, [rabbitmq has a STOMP
connector](http://www.rabbitmq.com/stomp.html) - we installed it on our
RabbitMQ server, and have it churning through both mcollective and logstash
requests just fine.

You can also code a different connector plugin for mcollective agents - I
haven't yet seen code out in the wild for connecting with an AMQP broker (or
anything else for that matter), but the option is out there, if you really
don't want to run a STOMP broker.

mcollective from tarball
-------------------------

There's no mcollective gem to be found anywhere, so in order to get a
consistent installation across all of our machines, we resorted to using the
tarball.  A couple of things to keep in mind if you choose to go this route:

- Be sure to set RUBYLIB to point to the lib directory in the tarball.
- Install the STOMP gem.
- If you're using the applications directly (as an mcollective client for
  example), add the unpacked mcollective directory to your PATH (or copy over
the Ruby binaries over to a bin folder of your choice.)

Configuration files
--------------------

It's not immediately obvious that you can split your configuration files into
multiple files, but *only* for plugin configuration.  If your server.cfg file
has entries such as "plugin.foo.bar = blah" and "plugin.bar.baz = 20", you can
write the following cfg files in /etc/mcollective/plugin.d instead of stuffing
those previous entries in server.cfg:

foo.cfg:

bar = blah

bar.cfg:

baz = 20

Again, it's not totally obvious from the documentation, but at least there's a
little bit of flexibility there for your configuration management system of
choice.

--

I'm quite happy with mcollective - it's easily extensible and it gives me much
better control over our UNIX machines.  We have it running in production and it
makes it a hell of a lot easier to inventory and control several machines at
once.
