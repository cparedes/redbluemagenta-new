---
title: Chef as a system integration tool, with a few examples.
layout: post
published: true
categories:
- Systems Administration
- Chef
- Configuration Management
date: 2010-12-02 16:40:00
---

I haven't been using configuration management tools for too long, but for
my particular use case (if I was directly managing a fleet of machines),
I am usually fine with just having an easier to use interface for managing
my machines.

However, I was tasked with building a Chef deployment that was to be a
"lights out", fully automated deploy that would not only setup the machines
automatically with their particular roles, but to also have said machines
gather system state from *other* machines and compute exactly what to do
based on other system states.  This required a lot of introspection: what
sort of things do I do when configuring machines to talk to each other?
Most of this is all second nature to me, and I tend to err on the side of
caution when configuring multiple services to talk to each other.  For
this particular project of which I have almost no experience in doing,
I figure that if I can at least write down my steps, I can automate those steps
in code.

Setting up MySQL machines is a good example of the differences between the
two methods of managing machines with a tool like Chef.  You can setup
my.cnf on each machine with Chef and have my.cnf be configured so that it
properly sets up MySQL to be a slave or a master depending on any attributes
that the user provides, along with any system specific tuning depending on
the output of ohai.  The additional steps required for the second method
is to actually setup replication automatically - this requires the system
to know exactly which systems are configured as masters, where the masters
are at in its binlogs, and what sort of credentials it needs in order
to communicate with the server.

In general, you must know your application before proceeding - as an example,
if you haven't worked with MySQL replication at all, then it's easy to
screw everything up, doubly so if you automate your ignorance into the code.

What helps for me is to first document exactly what steps I've taken to
setup said application, then document how the application ties with any
other service on the network.  Then redo your documentation in your favorite
configuration management system.  Figure out if you need each machine to
know every other machine's state; if so, then you might want to look into
chef-server, the Opscode platform, or Puppet + mcollective.

Once you've determined that you do need some sort of way to grab machine
states for each machine, you can now sit down and start writing your recipes
with the assumption that you'll be using chef-server or Puppet + mcollective.

Since I've been working with chef-server the most lately, I'll talk about
how I automated MySQL replication with chef-server.

In the recipes I've written, I've used the "search" feature of chef-server
quite extensively in order to gather collections of nodes that match certain
criteria.  For instance, I have two roles, "db-master" and "db-slave" which
set attributes on the nodes (that can be later pulled with ohai, knife, etc.)
that says exactly what database role that was applied to the machine.  For
instance, role[db-master] sets node[:mysql][:server][:role] = master, and
similarly, role[db-slave] sets node[:mysql][:server][:role] = slave.

*Figure 1: A slice of the role file for configuring the node's DB role.*
<pre><code>
...
default_attributes "mysql" => { "server" => { "role" => "slave", ... } }
...
</code></pre>

When the role is thrown into the run_list of a particular node, these
attributes will be set under mysql.server and will be stored by chef-server.

Now, let's say I want to collect all of the slaves that are known by
chef-server, so I can update the binlog positions/log file on each slave,
or at the very least, setup slave replication if it isn't already setup.
I use the search facility in chef-server in order to poll chef-server,
grab the results and stuff them into a variable.

*Figure 2: An example on using search in order to collect all of the DB slaves known to chef-server, then doing something to each of these nodes.*
<pre><code>
db_slaves = search(:node, 'mysql_server_role:slave')
db_slaves.each do |slave|
  # execute CHANGE MASTER TO query...
  # or maybe, puts "ohai #{slave[:fqdn]}!"
end
</code></pre>

Now, of course, we need a way to **know** which binlog position and log file
the slaves need to use from the master server.  Further, we need to allow
the replication user to connect to the master server for each slave machine
that's known by chef-server (you really don't want to blow open your
permissions with 'replication'@'%', someone can guess the password and
stream your data from your server.  Not a tractable solution.)  It ends
up that we can assign values to attributes in the recipes as well: we simply
grab the master status from the master, put the values into new attributes
that's pushed onto chef-server (and can be pulled with knife/ohai/etc.), and
then pull those attributes out again on the slave machines.

*Figure 3: Grab the master status from the master MySQL machine and throw them
into attributes, then on the slave machines, pull those attributes out again
and apply the changes to the slaves.*
<pre><code>
# Recipe snippet applied to master DB node
ruby_block "master-log" do
  block do
    logs = %x[mysql -u root -e "show master status;" | grep mysql].strip.split
    node[:mysql][:server][:log_file] = logs[0]
    node[:mysql][:server][:log_pos] = logs[1]
  end
  action :create
end

# Make sure that we create replication users for each slave machine known
# by chef-server.

slaves = search(:node, 'mysql_server_role:slave')

ruby_block "create-replication-users" do
  block do
    slaves.each do |slave|
      # slave_ip = <favorite mechanism to get IP address from hostname>
      %x[mysql -u root -e "GRANT REPLICATION SLAVE ON *.* TO 'replication'@'#{slave_ip}' IDENTIFIED BY 'password';]
    end
  end
  action :create
end

# Recipe snippet applied to slave DB nodes
masters = search(:node, 'mysql_server_role:master')

# Let's assume we only have one master for now
ruby_block "import-master-log" do
  block do
    # master_ip = <favorite mechanism to figure out an IP address \
                   from a hostname>
    master_log_file = masters[0][:mysql][:server][:log_file]
    master_log_pos = masters[0][:mysql][:server][:log_pos]
    %x[mysql -u root -e "CHANGE MASTER TO master_host='#{master_ip}', master_port=3306, master_user='replication', master_password='password', master_log_file='#{master_log_file}', master_log_pos=#{master_log_pos};"]
    %x[mysql -u root -e "START SLAVE;"]
  end
  action :create
end
</code></pre>

Of course, you should REALLY add in checks to make sure you don't accidentally
clobber your data - you really shouldn't be applying these things if the
machine already *has* the required info in place.  You'll definitely
want to add in checks to make sure that you don't apply master binlog
positions if the slave machine is already streaming updates from a master.

Another example is configuring a service that needs to have information of
multiple machines on the network; mysql-mmm is a great example of this.
As far as I know, with mysql-mmm conf files, you cannot have a conf.d folder
with individual configurations for each machine you want to provision, plus
you have defined sections in each mysql-mmm configuration file that cannot
easily be split apart.  You'll definitely need to use something that can
gather all of that info into a single place, then generate the conf file
based on some kind of central store's knowledge of the state of every machine.

If you guys have any questions on any of the above, don't hesitate to let me
know.

Hope this article helps!

**Further Reading:**

1. Chef Search Syntax - [http://wiki.opscode.com/display/chef/Search](http://wiki.opscode.com/display/chef/Search)
2. Solr - [http://lucene.apache.org/solr/](http://lucene.apache.org/solr/) (the search engine powering chef-server's search capabilities)
3. Slides from Orchestration Panel at Cloud Connect 2010 - [http://www.slideshare.net/dev2ops/orchestration-panel-at-cloud-connect-2010](http://www.slideshare.net/dev2ops/orchestration-panel-at-cloud-connect-2010)
