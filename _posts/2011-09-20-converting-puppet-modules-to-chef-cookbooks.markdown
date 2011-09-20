---
title: Converting Puppet Modules to Chef Cookbooks
layout: post
published: true
categories:
- Systems Administration
- Puppet
- Chef
- DevOps
date: 2011-09-20 00:05:00
---

Over a couple of caffeine induced nights, I've converted a handful of our
Puppet modules over to Chef cookbooks (by hand) to see how it'd all turn out.
We've not yet decided whether we will actually use Chef in production, but I
figure that if I've lowered the bar of entry, it would make the decision much
easier (we'd still have to worry about whether the technical merits justify
putting in time to transition everything over to Puppet, but at least the
initial cost of converting things over isn't as painful as it would've been.)

Here's a sample snippet of Puppet code that I'll be referring to for the rest
of the article:

<pre><code>
class foobar {
  $version = "1.0.0"
  $tarball = "foobar-${version}.tar.gz"
  $url     = "http://example.com/${tarball}"

  file { "/opt/foobar":
    ensure => directory,
    mode   => 0755,
  }

  exec { "download tarball":
    path    => "/bin:/usr/bin:/sbin:/usr/sbin",
    cwd     => "/opt/foobar",
    command => "wget ${url}",
    creates => "/opt/foobar/${tarball}",
  }

  exec { "extract tarball":
    path    => "/bin:/usr/bin:/sbin:/usr/sbin",
    cwd     => "/opt/foobar",
    command => "tar zxvf ${tarball}",
    creates => "/opt/foobar/foobar-${version}",
    before  => File["/opt/foobar/foobar-${version}/config.cfg"],
  }

  file { "/opt/foobar/foobar-${version}/config.cfg":
    content => template("foobar/config.cfg.erb"),
  }

  file { "/opt/foobar/foobar-${version}/staticfile":
    source => [ "puppet:///modules/foobar/staticfile.${hostname}",
                "puppet:///modules/foobar/staticfile" ],
  }
}
</code></pre>

Puppet resources port pretty well over to Chef resources - you simply have to
make sure that you're using the right Chef resource when rewriting it (for
example, you use "file" in Puppet for templates, regular files transferred from
the server, regular files managed only locally, and directories - these are all
separate things in Chef, with resources such as "cookbook_file", "template",
"file", "directory", and "remote_directory.")  As for variables, this might be
a bit of a judgment call - I've mostly thrown tunables into node attributes,
and leave temporary recipe-specific variables where they were in Puppet.  In
this case, I'd likely throw $version into "node[:foobar][:version]" as an
attribute - the tarball name, if the naming convention doesn't change between
versions, could probably stay inside the recipe itself.  The URL might be a
tunable too, and so could probably be stuffed into "node[:foobar][:url]" - what
if you had mirrors of these files in each geographical area?

Here's what our code will look like so far, given the above:

<pre><code>
# foobar/attributes/default.rb:
default[:foobar][:version] = "1.0.0"
default[:foobar][:tarball] = "foobar-#{node[:foobar][:version]}.tar.gz"
default[:foobar][:url]     = "http://example.com/#{node[:foobar][:tarball]}"
# Replaces the following Puppet code:
# $version = "1.0.0"
# $tarball = "foobar-${version}.tar.gz"
# $url     = "http://example.com/${tarball}"

# foobar/recipes/default.rb:

directory "/opt/foobar" do
  mode "0755"
end
# Replaces the following Puppet code:
# file { "/opt/foobar":
#   ensure => directory,
#   mode   => 0755,
# }
</code></pre>

Now, what do we do with the exec resources?  There's still a few things that
make sense to port directly to Chef's "execute" resources - things like
updating database users, running a program to alter configuration (think "make"
for creating sendmail database files), or extracting tarballs.  However, we do
have one exec resource that can be replaced by a regular Chef resource - the
exec resource that downloads a tarball onto local disk.  This is supplanted by
the "remote_file" resource - keep in mind that we likely do not want to keep
downloading the file on each Chef run (we guarantee this in Puppet with some
kind of command in the "onlyif" or "unless" statement - test -f? Current
downloaded file md5sum hash matches? etc.)  Probably the simplest way to deal
with this is to use a SHA256 hash in the remote_file resource - that's what
we'll do in this next code example:

<pre><code>
# foobar/recipes/default.rb:
remote_file "/opt/foobar/#{node[:foobar][:tarball]}" do
  source node[:foobar][:url]
  mode "0644"
  checksum "deadbeef"
end
# Replaces the following Puppet code:
# exec { "download tarball":
#   path    => "/bin:/usr/bin:/sbin:/usr/sbin",
#   cwd     => "/opt/foobar",
#   command => "wget ${url}",
#   creates => "/opt/foobar/${tarball}",
# }
</code></pre>

There isn't a way to do a simple existence check for the file before attempting
to download it (like what we've done in our Puppet code above), but this is a
bit more robust for dealing with remote files.

The tarball extraction exec resource in Puppet is easy to port over, and we'll
just go ahead and just do a straight translation over to Chef:

<pre><code>
# foobar/recipes/default.rb:

execute "extract tarball" do
  cwd "/opt/foobar"
  command "tar zxvf #{node[:foobar][:tarball]}"
  creates "/opt/foobar/foobar-#{node[:foobar][:version]}"
end

# We'll go ahead and throw in the template, too:

template "/opt/foobar/foobar-#{node[:foobar][:version]}/config.cfg" do
  source "config.cfg.erb"
end

# Replaces the following Puppet code:
# exec { "extract tarball":
#   path    => "/bin:/usr/bin:/sbin:/usr/sbin",
#   cwd     => "/opt/foobar",
#   command => "tar zxvf ${tarball}",
#   creates => "/opt/foobar/foobar-${version}",
#   before  => File["/opt/foobar/foobar-${version}/config.cfg"],
# }
#
# file { "/opt/foobar/foobar-${version}/config.cfg":
#   content => template("foobar/config.cfg.erb"),
# }
</code></pre>

Since Chef executes things in order, we don't have to specify that the tarball
extraction step must come before writing the template file - we simply place it
before the template creation step.

Also, Chef doesn't require you to specify a global "Path" variable in the top
level of your class hierarchy - it uses the PATH variable that's sourced in the
shell that's running Chef.  If you need to override this, you can by setting
the "path" attribute in the execute resource.

Lastly, dealing with the regular static file at the end of the Puppet module is
a bit simpler in Chef than in Puppet - sometimes, we need to have specific
files for specific hosts or operating systems, and the way we do this is by
throwing the file into different directories, instead of coming up with a
naming convention and trying to stick with it throughout Puppet module
development.  In this case, we have file specificity broken up by hosts,
and so we can create the following folders and files in our Chef cookbook:

<pre><code>
foobar/files/host-host1.example.com/staticfile
foobar/files/host-host2.example.com/staticfile
foobar/files/default/staticfile
</code></pre>

We then write the following in our recipe:

<pre><code>
# foobar/recipes/default.rb:

cookbook_file "/opt/foobar/foobar-#{node[:foobar][:version]}/staticfile" do
  source "staticfile"
end
# Replaces the following Puppet code:
# file { "/opt/foobar/foobar-${version}/staticfile":
#   source => [ "puppet:///modules/foobar/staticfile.${hostname}",
#               "puppet:///modules/foobar/staticfile" ],
# }
</code></pre>

All in all, it's not too tough to port things over to Chef - there's nothing
like [chef2puppet](https://github.com/relistan/chef2puppet) for converting
things automatically to Chef, but Chef resources map pretty well to Puppet
resources, and so the task isn't nearly as hard as it could have been.  Chef
has a bit more structure in place for stashing tunables separate from what
actually executes, but it could possibly be a bit more constraining than what
others could decide on with structuring their Puppet modules.

If anyone has any questions about this, don't hesitate to let me know via email
or through the comments section below.
