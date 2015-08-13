---
layout: post
title: "Elixir Regexes"
date: 2014-02-18 22:02:55 -0500
comments: true
categories: elixir, regex
---
I'm up to the OTP chapter in [Dave Thomas's Elixir book](http://pragprog.com/book/elixir/programming-elixir) now, but am taking a small break to tighten up my Elixir style.  I haven't read enough of other people's code.  I want to learn more of the idiomatic style of Elixir and functional languages, in general.

Among the tricks I have up that particular sleeve is [Exercism.io](http://exercism.io).  [I've written about Exercism.io before](http://variousandsundry.com/cs/blog/2013/09/05/exercism-dot-io-first-thoughts/), but just to recap: You are presented a series of programming challenges with all the tests laid out for you.  You can use TDD to develop the program, then upload it and have others comment on it.

In other words, here's a chance to see a lot of other people's Elixir code and have mine looked at by others.  Exciting.

The first programming exercise, named "Bob", deals with regular expressions.  Great, I think.  I'm a Perl programmer.  I've got regexes in the bag.  Yet I'm barely 3 tests into the 12 test exercise, and I've already learned a lot about pattern matching, ```case```, ```cond```, and regexes in Elixir.

The first tip to know is this: The latest and greatest releases of Elixir use a tilde (~), not the percent sign. If you're using 0.12.3 on your computer and [the documentation on-line is for 0.12.5-dev](http://elixir-lang.org/docs/master/), you're a couple of versions behind and might want to catch up. The percent was needed elsewhere for the soon-to-come ```map``` functionality that will wipe records from the face of the language.

For example, what once was this:

	Regex.run(%r/\w.*/, string)

is now this:

	Regex.run(~r/\w.*/, string)

One character, a world of difference.  This is the fun of learning a language as it's being developed.  You think things are settled, but then a single character change has you spinning in circles for far too long.c
