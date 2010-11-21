---
title: Bootstrapping Chef Server on CentOS 5
layout: post
published: false
categories:
- Systems Administration
- Chef
- CentOS
- Configuration Management
---

After learning [Chef] over the past two weeks and struggling to figure out
how to bootstrap a Chef server on a CentOS 5 system, I've figured out
the exact steps to go from a raw machine to a working Chef server with
certificates installed, a caching proxy server, etc.

Our goals are going to be the following:

1. Setup chef-server to a point where we can interact with the system
with `knife` on our local machine.
2. Setup apache as a caching reverse proxy listening on port 443/444 for
the API server and (optionally) the web interface respectively.

First of all, make sure you have the EPEL repository setup.  There are
instructions on how to do this on the [Fedora Project wiki].

You will want to install couchdb, rabbitmq-server, java-1.6.0-openjdk,
zlib-devel, and libxml2-devel.  The last two packages are necessary
for building native extensions for the set of chef gems we're going to
be installing later in the guide.

<pre><code>
# yum -y install couchdb rabbitmq-server java-1.6.0-openjdk zlib-devel libxml2-devel
</code></pre>

You'll also likely want to install [REE 1.8.7].  Chef doesn't run on the
system provided Ruby on CentOS 5.5 or below, so you'll have to upgrade
Ruby.  You can, alternatively, define the RSTRING_PTR and RSTRING_LEN macros
in /usr/lib/ruby/1.8/(architecture)-linux/ruby.h along the top of the file:

<pre><code>
...
#define RSTRING_PTR(s) (RSTRING(s)->ptr)
#define RSTRING_LEN(s) (RSTRING(s)->len)
...
</code></pre>

I don't recommend doing this however, given that the issue is properly fixed in
subsequent versions of Ruby and that Chef is only officially supported on Ruby 1.8.7 and above.

After getting REE 1.8.7 installed, you'll now want to install the rubygems
associated with the whole set of Chef packages.

<pre><code>
# gem install chef chef-server chef-server-api chef-solr chef-server-webui --no-ri --no-rdoc
</code></pre>

You can leave out "chef-server-webui" if you don't want to use the web
management interface.

Create a chef/chef user/group:

<pre><code>
# useradd chef
</code></pre>

The gem installation doesn't touch anything outside of the gems directory,
and since chef requires access to /var/log, /etc, etc., we'll have to create
these directories ourselves.

<pre><code>
# mkdir -p /srv/chef
# chown chef:chef -R /srv/chef
# mkdir -p /etc/chef/certificates
# chmod 777 /etc/chef/certificates
# chown chef:chef /etc/chef/certificates
# mkdir -p /var/log/chef
# chown chef:chef -R /var/log/chef
# mkdir -p /var/run/chef
# chown -R chef:chef /var/run/chef
# mkdir -p /var/chef
# chown -R chef:chef /var/chef
</code></pre>

You'll likely want to symlink the chef binaries from $GEM_DIR/gems/chef{-server,-solr,...}-0.9.12/bin to /usr/local/bin or /usr/bin; you don't have to though, this is mostly so that the init scripts work as provided by the chef-0.9.12 gem.

Copy over all of the contents in $GEM_DIR/gems/chef-0.9.12/distro/<distro> to their respective directories in the root directory of the machine.  There's also man pages that you can copy over from $GEM_DIR/gems/chef-0.9.12/distro/common, though they're largely out of date - you're better off using knife --help for more up to date documentation.



[Chef]: http://opscode.com
[Fedora Project wiki]: http://fedoraproject.org/wiki/EPEL
[REE 1.8.7]: http://www.rubyenterpriseedition.com/
