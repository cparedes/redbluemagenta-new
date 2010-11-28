---
layout: default
title: root
---

You've reached Christian "Ian" Paredes's website.

I'm a senior system administrator at [Blue Box Group].  This is my personal
technical blog.  I write about system administration related topics, as well
as other things of interest, such as programming, music, and general
advocacy for various technology related issues.

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

[Blue Box Group]: http://blueboxgrp.com
[all blog posts]: /archive.html
