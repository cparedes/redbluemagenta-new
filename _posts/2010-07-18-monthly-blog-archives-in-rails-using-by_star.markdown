--- 
layout: post
title: Monthly blog archives in Rails using by_star
post_id: "631"
categories:
- By_star
- Enki
- Programming
- Rails
- Ruby
---
<em>(EDIT: Credit for part of the show method goes to <a href="http://stackoverflow.com/questions/2726525/group-and-sort-blog-posts-by-date-in-rails">Senthil on Stack Overflow</a>.)</em>

I've finally gotten monthly archives to work in <a href="http://enkiblog.com">Enki</a>, which is a minimalist blog engine built on top of Rails; you can most likely follow these directions with whatever Rails blogging app that you have built from scratch.

Basically, how this works is that when the user visits /archives/:year, all of the posts are enumerated for that specific year; if the user visits /archives/:year/:month, all of the posts for that specific month are shown; and finally, if the user visits /archives/:year/:month/:day, all of the posts for that specific day are shown.

First of all, in Enki, there's a separate Archives controller and route already created for you; instead of breaking convention, I've decided to extend the controller by adding the magic in the "show" method of the controller.  Further, I've decided to use the by_star gem, in order to easily collect posts by year, by month, and by day.

Before doing this, be sure to install the by_star gem:

<pre class="brush: shell">
$ gem install by_star
</pre>

Here's what the controller looks like:

<pre class="brush: ruby">
class ArchivesController < ApplicationController
...
def show
    year = params[:year]
    month = params[:month]
    day = params[:day]
    
    if (year && month && day)
      @entries = Post.by_day(Time.local(year, month, day)).reverse
    elsif (year && month && !day)
      @entries = Post.by_month(Time.local(year, month)).reverse
    else
      @entries = Post.by_year(year).reverse
    end
  end
end
</pre>

Here's what my routes.rb file looks like:

<pre class="brush: ruby">
...
  map.archives '/archives', :controller => 'archives', :action => 'index'
  map.connect '/archives/:year/:month/:day', :controller => 'archives', :action => 'show'
  map.connect '/archives/:year/:month', :controller => 'archives', :action => 'show'
  map.connect '/archives/:year', :controller => 'archives', :action => 'show'</code>
</pre>

The next thing I'm working on is the page that's displayed when the user visits /archives/; the page should only display the years that actually have posts, and after the user clicks the year, it'll drop down a list of months that have posts.
