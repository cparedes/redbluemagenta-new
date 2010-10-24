--- 
layout: post
title: Social Media Wishlist
post_id: "222"
categories:
- Computers
- Cpan
- Facebook
- Mediawiki
- Mvs
- Programming
- Vi
- Wordpress
---
I've been thinking about this just now: how awesome would it be to able to update my Wordpress site, a MediaWiki site, Facebook, etc. all in the command line (without using something as unwieldy as Emacs? :D)  I'm sure this is possible; in fact, I know it's possible with MediaWiki (WWW::Mediawiki::Client in CPAN provides a command line client called "mvs".)  However, it'd be great to, for instance, have a directory in /var/ or /home/user/sites/ for each social media site (like for instance, /home/user/sites/wp for Wordpress, /home/user/sites/mediawiki for MediaWiki, etc.)  Each of these directories would have flat text files containing new posts to add.  Then I could run a shell script that pushes these text files onto each respective site.

I'll have to sketch this out sometime, this could be an interesting project for me to work on.
