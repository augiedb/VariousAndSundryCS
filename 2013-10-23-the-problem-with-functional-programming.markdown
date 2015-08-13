---
layout: post
publish: true
title: "The Problem with Functional Programming"
date: 2013-10-23 20:02
comments: true
categories: 
---

Functional programming does certain things very well.  Amazingly well.  Blows other programming out of the water.

Other things, I don't know that it does much better.  That limits it.  And the problem is, unless a language can be all things to all people these days, why bother switching to it, or adding it as a major new tool in the tool belt?  Being intellectually curious about something is one thing, evangelizing it in your enterprise career is another.  Does it make much sense to start using a new language just to create HTML tables from database stores?  Ruby, PHP, Perl, etc. will do that just fine.

So what's the killer app for Erlang, Haskell, Elixir, etc. that blog writing was for Ruby on Rails, for example?

[Finn Espen Gundersen nails it in his blog:](http://www.gundersen.net/functional-programming-hurdle-uninteresting-programs/)

>...if we want to convey the usefulness of FP [functional programing] to the imperatives amongst us, we need to focus on elegant solutions to real world inputs and outputs. Most software needs to integrate with other software. We need to show how our I/O-libraries, often overlooked or thought of as mere &#8220;helpers&#8221;, are superior in efficiency and leads to cleaner and more maintainable code. While a pure HashMap is a beautiful thing, let&#8217;s focus more of our time on Haskell packages such as <a title="postgresql-simple" href="http://hackage.haskell.org/package/postgresql-simple" target="_blank">postgresql-simple</a>, <a title="amqp" href="http://hackage.haskell.org/package/amqp" target="_blank">amqp</a> (of which I am a minor contributor), <a title="aeson" href="https://www.fpcomplete.com/school/text-manipulation/json" target="_blank">aeson</a> and the fact that there is no native MSSQL library, just ODBC. Let&#8217;s focus on the plumbing, so to say. Great packages for integrating with the outside world is what will bring FP to the masses. When that prerequisite is there, only then are they ready for all the other stuff we have.