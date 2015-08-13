---
layout: post
title: "Immutable Infrastructure"
date: 2014-08-06 10:20:57 -0400
comments: true
categories: 
---
[Chad Fowler nails this one.](http://chadfowler.com/blog/2013/06/23/immutable-deployments/) It's a year old, but still on point:

>Cron jobs spring up in unexpected places, running obscure but critical functions that only one person knows about. Application code is deployed outside of the normal straight-from-source-control process.

>The system becomes finicky. It only accepts deploys in a certain manual way. The init scripts no longer work unless you do something special and unexpected.

>And, of course the operating system has been patched again and again (in the best case) in line with the standard operating procedures, and the inevitable entropy sets in. Or, worse, it has never been patched and now you’re too afraid of what would happen if you try.

>The system becomes a house of cards. You fear any change and you fear replacing it since you don’t know everything about how it works.

I especially like this post on a week where I'm trying to configure a server at work to match current servers we already have and, well, it hits home a little too hard...
