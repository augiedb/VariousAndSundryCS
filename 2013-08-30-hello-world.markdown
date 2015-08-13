---
layout: post
title: "Hello, World"
date: 2013-08-30 22:09
comments: true
categories: meta 
---
This is a first test of my new Octopress-driven blog to discuss the finer points of computer programming geekery.

I'm a Perl programmer by trade, but have spent a lot of time in the last year learning Ruby, Ruby on Rails, Elixir, and iOS programming.  Call it a programming mid-life crisis, if you will.

I've started this blog to ramble on about the things that interest me in that world but that regular readers of the main blog, [VariousAndSundry.com](http://www.variousandsundry.com), which is so old in internet terms that I still have the "www." in front of it.

Just to test how this blog handles code excerpts, here's a little something I wrote last night in Elixir:

``` ruby Part of a Classic Guessing Game
defp guess(actual, low.._high, c_guess) when c_guess > actual do
  IO.puts "Is it #{c_guess}"
  new_range = (low..c_guess)
  guess(actual, new_range, take_a_guess(new_range))
end
```

Frustratingly, code highlight for Elixir is not available in pygments.rb.  It *is* available in [the main Python-based pygments highlighting system](https://bitbucket.org/birkenfeld/pygments-main/pull-request/57/add-elixir-and-elixir-console-lexers/diff), so perhaps we'll see it ported over someday soon...
