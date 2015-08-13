---
layout: post
title: "Ruby: Inheritance is Super"
date: 2013-11-15 20:57
comments: true
categories: ruby
---

Let me save you all an hour with this one:

When using inheritance in Ruby, the parentheses in your call to ```super``` are important.

Specifically: If you're calling out to ```super``` without any parameters, you *must* use them.

This is a lesson I have learned now the hard way, after troubleshooting seemingly every other potential problem in my code. I was convinced at one point that one class couldn't see the other because it was in a separate file. I messed with "require_relative" for a while, plus different code layout choices.

In the end, the problem came about becuase I was inheriting from another class, but not passing additional values over as a parameter.  Because of that, I had to use the parentheses.  Once I realized that, problem solved.

So, yes, I feel like a bit of a fool on this, but I also just learned a lot about:

 - require_relative
 - the ```super``` command
 - method inheritance

Sometimes, learning is a slow slog. For more on learning, check out [the latest edition of the Ruby Rogues podcast.](http://rubyrogues.com/131-rr-how-to-learn/)
