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

Now, make every Chef init script in /etc/init.d executable:

<pre><code>
# chmod +x /etc/init.d/chef-*
</code></pre>

then enable each service:

(Debian)

<pre><code>
# for svc in solr solr-indexer server server-webui ; do update-rc.d chef-${svc} defaults ; done
</code></pre>

(CentOS/RHEL)

<pre><code>
# for svc in solr solr-indexer server server-webui ; do chkconfig --add chef-${svc} ; chkconfig chef-${svc} on ; done
</code></pre>

For apache2's configuration for chef-server and chef-server-webui, I've simply used a heredoc to write out the configuration which autofills exactly where the document root should be (since it depends on where your gems are located.)

<pre><code>
cat > /etc/httpd/conf.d/chef-server.conf <<EOF
<VirtualHost *:443>
  ServerName $(hostname)
  DocumentRoot "$(echo $GEM_PATH | awk -F":" '{print $1}')/gems/chef-server-api-0.9.12/public"
  
  <Proxy balancer://chef_server>
    BalancerMember http://127.0.0.1:4000
    Order deny,allow
    Allow from all
  </Proxy>
  
  LogLevel info
  ErrorLog /var/log/httpd/chef_server-error.log
  CustomLog /var/log/httpd/chef_server-access.log combined
   
  SSLEngine On
  SSLCertificateFile /etc/chef/certificates/$(hostname).pem
  SSLCertificateKeyFile /etc/chef/certificates/$(hostname).pem
  
  RequestHeader set X_FORWARDED_PROTO 'https'
  
  RewriteEngine On
  RewriteCond %{DOCUMENT_ROOT}/%{REQUEST_FILENAME} !-f
  RewriteRule ^/(.*)$ balancer://chef_server%{REQUEST_URI} [P,QSA,L]
</VirtualHost>
  
<VirtualHost *:444>
  ServerName $(hostname)
  DocumentRoot "$(echo $GEM_PATH | awk -F":" '{print $1}')/gems/chef-server-webui-0.9.12/public"
  
  <Proxy balancer://chef_server_webui>
    BalancerMember http://127.0.0.1:4040
    Order deny,allow
    Allow from all
  </Proxy>
  
  LogLevel info
  ErrorLog /var/log/httpd/chef_server-webui-error.log
  CustomLog /var/log/httpd/chef_server-webui-access.log combined
   
  SSLEngine On
  SSLCertificateFile /etc/chef/certificates/$(hostname).pem
  SSLCertificateKeyFile /etc/chef/certificates/$(hostname).pem
  
  RequestHeader set X_FORWARDED_PROTO 'https'
  
  RewriteEngine On
  RewriteCond %{DOCUMENT_ROOT}/%{REQUEST_FILENAME} !-f
  RewriteRule ^/(.*)$ balancer://chef_server_webui%{REQUEST_URI} [P,QSA,L]
</VirtualHost>
EOF
</code></pre>

Now we're ready.  Start the daemons in the following order:

1. chef-solr
2. rabbitmq-server
3. chef-solr-indexer
4. couchdb
5. chef-server
6. chef-server-webui

Between starting (2) and (3), you will have to add rabbitmq vhosts and users, as well as set permissions on the vhost you create.

<pre><code>
# rabbitmqctl add_vhost /chef
# rabbitmqctl add_user chef password
# rabbitmqctl set_permissions -p /chef chef ".*" ".*" ".*"
</code></pre>



[Chef]: http://opscode.com
[Fedora Project wiki]: http://fedoraproject.org/wiki/EPEL
[REE 1.8.7]: http://www.rubyenterpriseedition.com/
