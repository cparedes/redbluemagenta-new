---
layout: default
title: root
---

<center><a href="https://www.eff.org/pages/say-no-to-online-censorship"><img src="https://w2.eff.org/images/no_censorship_button.jpg"></a></center>

*I'm placing this image prominently on the front page of my site to show my support for a free internet: a place without any restrictions on free speech by any governments or corporations.*

You've reached Christian "Ian" Paredes's website.

I'm a senior system administrator at [Blue Box Group].  This is my personal
technical blog.  I write about system administration related topics, as well
as other things of interest, such as programming, music, and general
advocacy for various technology related issues.

Looking for Sara?  [Here's her page].  She's awesome.  She can probably kick
my ass in Python programming.

Now in IPv6 with SixXS's [Tunnel Broker](http://sixxs.net), IP address 2001:1938:81:1ce::2!

10 Most Recent Posts
--------------------

<ul>
{% for post in site.posts limit:10 %}
  <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>

[all blog posts]

Want to reach me?

**IRC**: cparedes @ irc.freenode.net, #lopsa, #gslug, #sasag, #chef, #blueboxgroup

**GTalk**: cp@redbluemagenta.com

[Blue Box Group]: http://bluebox.net
[all blog posts]: /archive.html
[Here's her page]: http://sara.redbluemagenta.com/
