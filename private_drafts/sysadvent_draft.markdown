---
layout: post
title: Chef as a system integration tool, with more examples.
---

(Note: This is a followup to my blog post that I've posted on my
[personal blog].  We've covered some basics in automatically setting up MySQL
replication with Chef, now we're going to cover a few more examples in
setting up application servers that automatically grab information on
what DB machines exist in the network, and setup the application accordingly.)

Setting up application servers with chef-solo or Puppet with information on
which machines to connect to is easy enough - we can simply define a new node
definition with proper attributes pointing to the machines we know to be
whatever role we've configured the other machines as.  The rough deployment
plan, in this case, goes something like this:

1. Determine hostnames of would be data stores, caching servers, etc.
2. Configure machines with those hostnames and the services that they require
to operate as a data store, caching server, etc.
3. Determine hostname of a new app server.  Make sure that the app server can
access the data stores, caching servers, etc.
4. Configure app server and ensure that it knows which machines to connect to
for the various services it depends on.

We can automate most of everything as far as getting hostnames configured,
getting services up and running on each individual box, etc.

For configuring permissions on all appropriate services so that a new app
server can connect to each machine, or even configuring an app server to
know which servers to connect to, either requires a priori knowledge on the
part of the sysadmin, or at the very least, a central store of system state
that can be polled and operated on. (The sysadmin is thus replaced with an
automaton. :))

As with the previous article, we look to solutions such as Puppet
+ mcollective and chef-server; we can poll for certain metadata from a central
store, grab machine names that fulfill the criteria, and execute an action
against those machines.

With Puppet + mcollective or chef-server, the deployment plan now looks
roughly like this:

1. Pick a machine, apply a role to the machine.  We'll call the role "role-1".
2. Continue doing this for however many machines you want to have that specific
role.
3. Pick another set of machines, apply a role that is contingent on role-1.
We'll call this "role-2".
4. role-2, during its execution, will figure out which machines are using
the role "role-1".

We do not have to refer to any particular hostnames or any particular sets
of machines; instead, we rely on metadata and construct sets of machines
based around more descriptive metadata, rather than having to point at any
particular machine with a specific hostname.

Let's assume we have a fleet of machines running MySQL, with one machine
acting as a master, and a whole bunch of other machines acting as slaves.
Let's also assume that we're deploying a Rails app and that we've mostly
got code deployments down (we just need to configure the Rails app to
connect to whatever DB machines happen to be known by chef-server.)

First, we spin up the app server.  We tag it as an app server via some sort
of attribute that can be searched later on by chef; we apply this attribute
via a role file.

*Figure 1: A slice of a role file that sets a default attribute for the machine.*
<pre><code>
...
default_attributes "app" => true
...
</code></pre>

(We can also search against what roles are applied to the machine; for example,
if we've applied the role "role[app-server]", we can search for
"roles:app-server" in our recipe.)

After spinning up the app server with the applied attribute, we can now
search for it in our chef recipes.  Since we've assumed that the data stores
are already spun up, presumably they already have attributes applied to them
that would identify them as data store machines; thus, we'll assume that we
can search against those attributes and pull out the names of the data stores
that are known by chef-server.

Since you're a good sysadmin and haven't added any users to MySQL that can
be accessed by every host, we can assume that we'll be searching for app
servers that must be able to access the data store.  So we'll go ahead
and sketch out the code that does this.

*Figure 2: Search for app servers known by chef-server, then add appropriate permissions in MySQL for the app servers.*
<pre><code>
# Execute only on master servers!
app_servers = search(:node, "app:true")
app_servers.each do |app_server|
  server_ip = app_server[:ipaddress]
  ruby_block "add-#{app-server[:fqdn]}-permissions" do
    block do
      %x[mysql -u root -e "grant select,insert,update,... on #{db} to '#{db_user}'@'#{server_ip}' identified by '#{db_password}';]
    end
    not_if "mysql -u root -e \"SELECT user,host FROM mysql.user\" | grep #{db_user} | grep #{server_ip}"
    action :create
  end
end
</code></pre>

Okay, now that's done.  Now we have to make sure that the app server knows of
which data store machines are masters and which ones are slaves.  This time,
we'll have to have a template of our database.yml file stowed away in our
recipe that can be populated with database information.

*Figure 3: database.yml.erb example.*
<pre></code>
# We'll assume we're using this gem: https://github.com/schoefmax/multi_db

production:
  adapter: mysql
  database: <%= @db %>
  username: <%= @db_user %>
  password: <%= @db_password %>
  host: <%= @db_master %>

<% iterator = 1 %>
<% @db_slaves.each do |slave| %>
production_slave_database_<%= iterator %>:
  adapter: mysql
  database: <%= @db %>
  username: <%= @db_user %>
  password: <%= @db_password %>
  host: <%= slave %>
<% iterator += 1 %>
<%- end %>
</code></pre>

Then we refer to our template and generate it based on the results of our
search against all of the data store machines:

*Figure 4: Find all data store machines, determine which ones are masters and slaves, then throw results into variables that are used in our database.yml.erb template.*
<pre><code>
masters = search(:node, 'mysql_server_role:master')
slaves = search(:node, 'mysql_server_role:slave')
slaves_fqdn = slaves.collect { |x| x[:fqdn] }

template "/path/to/rails/config/database.yml" do
  source "database.yml.erb"
  mode 0644
  owner "foobar"
  group "foobar"
  variables(
    :db => db,
    :db_user => db_user,
    :db_password => db_password,
    :slaves => slaves_fqdn,
    :db_master => masters[0][:fqdn]
  )
  action :create
end
</code></pre>

We can add more magic in the template declaration in order to punt Passenger
or Mongrel so that the new code goes into effect.

In any case, the biggest win in automating system integration tasks is that it
not only cuts down time spent in doing the final set of steps in setting up
services with each other, but it also helps look at machine provisioning in
a different light - no longer do I care which particular machine is a DB/app
server, now all I worry about is that I have five app servers and two DB
servers.

Anyway, hope this article helps!

Further Reading:
================

1. [http://redbluemagenta.com/2010/12/02/chef-as-a-system-integration-tool/](http://redbluemagenta.com/2010/12/02/chef-as-a-system-integration-tool/) - Chef system integration article from my blog.
2. [http://wiki.opscode.com/display/chef/Search](http://wiki.opscode.com/display/chef/Search) - Chef Search syntax and other usage examples.
3. [http://www.slideshare.net/dev2ops/orchestration-panel-at-cloud-connect-2010](http://www.slideshare.net/dev2ops/orchestration-panel-at-cloud-connect-2010) - Slides from Orchestration Panel at Cloud Connect 2010.

[personal blog]: http://redbluemagenta.com/2010/12/02/chef-as-a-system-integration-tool/
