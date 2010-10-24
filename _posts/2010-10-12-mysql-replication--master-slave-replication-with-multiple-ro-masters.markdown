---
layout: post
title: MySQL Replication - master slave replication with multiple R/O masters
categories:
- Systems Administration
- Databases
- MySQL
date: 2010-10-12 20:41:00
---

(NOTE: I know, the blog post says that I'm working with multiple R/O masters.
The guide doesn't tell you how to do such a thing.  However, the principles
are quite the same, and you can easily extend this guide to make one of your
other R/O slaves to be an R/O master.)

So, after battling MySQL for a couple of days, I've come out with a bit more
experience with replication and a few pitfalls that I've encountered that you
may encounter as well.

The MySQL installation I was wrestling with consisted of a single R/W master,
a couple of R/O slaves, a R/O slave that also acted as an R/O master, and
several R/O slave machines slaving off of the R/O master.

Apologies for the ASCII art:

<pre><code>
                 * - R/W Master
                 |
                /|\
               / | \
 R/O slave  - *  *  * - R/O slave
                 |\_ R/O slave/master
                 |
                /|\
               * * * - R/O slaves
</code></pre>

We assume that the R/W master already has a bunch of data, and that we need
to copy all of the data over to each machine.

The biggest thing to keep in mind is that it's REALLY REALLY easy to mess
this up - if your replication settings are off by just a little bit on
the slave machines, then you're going to corrupt your data on the slave
machines.

Also, when we do the data copy to each machine, we will want to wipe out any
relay logs, any master binary logs, the master.info file, and the
slave-relay-bin.info file on the slave machines.  We'll recreate the
master.info file through the MySQL CLI.

So, without further ado, here's how you set everything up:

Step 1: Create a backup and ship it off to all of the slave machines.
---------------------------------------------------------------------

You'll have to make sure that either an existing slave server, or your master
server, either has a lock on all tables (for mysqldump), or the daemon is
switched off completely (for raw data backup with tar.)  In either case,
you'll need to make sure you can read all of the data without it being written
to at the same time.

In my particular case, I've shut off the daemon and grabbed a snapshot using
tar, since I had a spare slave server with an up to date copy of the data.
I streamed the compressed tar data over SSH to the slave servers:

<pre><code>cd /var/lib ; tar -czf - mysql | \
ssh <host> "tar -C /destination/folder -zxvf -" </code></pre>

Depending on how large your database is, it may or may not be popcorn time. :)

Step 2: Remove existing relay logs, master.info, etc.
-----------------------------------------------------

By default, these files are stored in /var/lib/mysql.

Remove all of them:

<pre><code>cd /var/lib/mysql ; rm mysql-bin* slave-relay-bin* \
relay-log* mysqld-relay-bin* master.info</code></pre>

Step 3: Setup my.cnf on the master and all slave machines.
----------------------------------------------------------

Okay, so I know this is pedantic, but we know for sure how to identify
different machines from each other in some way or another, either by an
IP address, a hostname, or some other metadata.  MySQL uses an internal
identification scheme for identifying different machines in an MySQL
replication environment.  You can define this ID under the [mysqld] section
in my.cnf (usually located in /etc/mysql/my.cnf), using the configuration
option "server-id".  These must be unique for every machine in the entire
set of machines that's participating as either a master or a slave in
the replication setup.

I usually start my numbering from the master, and work my way down.  For
instance, I'll number my R/W master as server-id = 1, my R/O master as
server-id = 2, one of the slaves as server-id = 3, etc.

We'll split this step into two parts: first part will be for master
machines (R/W and R/O), and R/O slave machines.

R/W and R/O masters:

You'll want to make sure that the following are defined in my.cnf:

log-bin = /var/log/mysql/mysql-bin.log  
log-bin-index = /var/log/mysql/mysql-bin.index  
log-slaves-updates  

For the R/O master, you'll want to also set read_only = 1.

R/O slave:

relay-log = /var/log/mysql/slave-relay.log  
relay-log-index = /var/log/mysql/slave-relay.index  
relay-log-info = /var/log/mysql/slave-relay.info  
replicate-ignore-db = mysql  
read_only = 1

One more note: make sure skip-slave-start is set on all slave machines in
my.cnf for now; we'll delete the line later, but this is necessary so we
can have as much time as possible in setting up the slave machines.

Step 4: Setup MySQL replication on all slaves.
----------------------------------------------

We're now ready to set up MySQL replication on all of the slave machines.

We're going to set up grants on the R/W master to the two slave machines
and the R/O master, and then we're going to set up grants on the R/O
master for each of its slaves.

Let's assume that the IP address scheme is as such:

<pre><code>
R/W Master: 10.0.0.1
|_ R/O Slave: 10.0.0.2
|_ R/O Slave: 10.0.0.3
|_ R/O Master: 10.0.0.4
 \_ R/O Slave: 10.0.0.11
 |_ R/O Slave: 10.0.0.12
 |_ R/O Slave: 10.0.0.13
</code></pre>

Alright, let's login to the R/W master and grant replication rights to its
three slaves:

<pre><code>rwmaster# mysql -u root
mysql> GRANT REPLICATION SLAVE ON *.* TO \
'replicate'@'10.0.0.2' IDENTIFIED BY 'foobar';
mysql> GRANT REPLICATION SLAVE ON *.* TO \
'replicate'@'10.0.0.3' IDENTIFIED BY 'foobar';
mysql> GRANT REPLICATION SLAVE ON *.* TO \
'replicate'@'10.0.0.4' IDENTIFIED BY 'foobar';
mysql> FLUSH PRIVILEGES;
mysql> \q</code></pre>

Stop the master's daemon:

<pre><code>rwmaster# /etc/init.d/mysql stop</code></pre>

Now, let's login to the R/O master and grant replication rights to its three
slaves:

<pre><code>romaster# mysql -u root
mysql> GRANT REPLICATION SLAVE ON *.* TO \
'replicate'@'10.0.0.11' IDENTIFIED BY 'barfoo';
mysql> GRANT REPLICATION SLAVE ON *.* TO \
'replicate'@'10.0.0.12' IDENTIFIED BY 'barfoo';
mysql> GRANT REPLICATION SLAVE ON *.* TO \
'replicate'@'10.0.0.13' IDENTIFIED BY 'barfoo';
mysql> FLUSH PRIVILEGES;
mysql> \q</code></pre>

We grab the master status from the R/W master and the R/O master, and note
it down for later usage:

<pre><code>rwmaster# mysql -u root
mysql> SHOW MASTER STATUS\G
...
File: mysql-bin.log.000500
Position: 5150150
...

romaster# mysql -u root
mysql> SHOW MASTER STATUS\G
...
File: mysql-bin.log.000001
Position: 4
...
</code></pre>

Again, note these down.  The R/W master binlog file and position will be
needed for its immediate slaves and the R/O master, and the R/O master file
and position will be relevant for its immediate slaves.  Also note down
the passwords and username for each grant: we're going to use this information
to connect to the respective masters in order to start replication.

**YOU CANNOT USE THE SAME BINLOG FILE AND INFORMATION FOR ANY SLAVES NOT
DIRECTLY CONNECTED TO THE MASTER.**  If you do, expect replication to break.
I've learned this the hard way.  Always assume that any replication you're
setting up is relevant only for any direct connection between a master/slave.

With these binlog files and positions in mind, you'll now set up each slave
to connect to its direct master.

We're going to start with the R/W master's direct slaves.

Connect to each slave and enter in the following information:

<pre><code>mysql> CHANGE MASTER TO \
MASTER_HOST='10.0.0.1', \
MASTER_PORT='3306', \
MASTER_LOG_FILE='mysql-bin.log.000500', \
MASTER_LOG_POS='5150150', \
MASTER_USER='replicate', \
MASTER_PASSWORD='foobar';
</code></pre>

where MASTER_LOG_FILE and MASTER_LOG_POS is grabbed from the output of
'SHOW MASTER STATUS\G' on the R/W master.

Now, connect to the R/O slaves that are connected directly to the R/O slave,
connect to the MySQL daemon, then type in the following:

<pre><code>mysql> CHANGE MASTER TO \
MASTER_HOST='10.0.0.4', \
MASTER_PORT='3306', \
MASTER_LOG_FILE='mysql-bin.log.000001', \
MASTER_LOG_POS='4', \
MASTER_USER='replicate', \
MASTER_PASSWORD='barfoo';</code></pre>

In order to be extra cautious about the data on each slave, we want to start
from the least significant slaves (the ones attached to the R/O master, since
they won't receive their updates until the R/O master receives its updates
from the R/W master), and then work our way up until we get to the R/W master.
(Reason why we're being paranoid about this is because it's very possible, if
we worked in the opposite direction, for the R/W master to get too ahead of
the slaves for us to actually set up the slaves for replication.  In this
case, we'd have to start over from scratch and create a new snapshot of the
data.  Not good.)

Go to the R/O master's slaves and type the following in:

<pre><code>mysql> START SLAVE;
</code></pre>

Then go to the R/O master and the R/W master's slaves and type the following
in:

<pre><code>mysql> START SLAVE;
</code></pre>

Now start the R/W master's daemon back up:

<pre><code>rwmaster# /etc/init.d/mysql start</code></pre>

You can run 'SHOW SLAVE STATUS;' on each slave machine to see if it's
replicating normally.

If not, you can check /var/log/mysql/mysqld.log (or mysqld.err) and see
what's breaking.

That's it!

Let me know if this guide has been helpful for you guys. :)
