---
layout: post
title: "Elixir: Making Apple Developers Feel At Home"
date: 2014-07-12 00:16:02 -0400
comments: true
categories: Elixir, Objective-C
---

Welcome, all you fancy iOS and Mac programmers.  Elixir wants __you__!  Why bother with Swift when Elixir already has the pipeline operator that so many are filing [Radars](http://fixradarorgtfo.com) requesting?  ([Or hack one of your own.](http://tetontech.wordpress.com/2014/06/11/swift-operator-overloading-and-chaining-functions-and-closures/))

Here, sit right back and I'll make you feel at home.

Elixir has structs. You're C-type folks, so you have to love that.  They're pretty easy to implement:

		$ vi comics.exs

		defmodule Comics do
			defstruct title: "", issue: 1, price: 3.99
		end

To create a comic, just give it whatever details you have, and the defaults will fill in the rest:

		$ iex comics.exs
		Erlang/OTP 17 [erts-6.1] [source] [async-threads:10] [kernel-poll:false]

		Interactive Elixir (0.14.3-dev) - press Ctrl+C to exit (type h() ENTER for help)

		iex> c1 = %Comics{title: "Asterix", issue: 10, price: 9.95}
		%Comics{issue: 10, price: 9.95, title: "Asterix"}

		iex> %Comics{title: "Asterix", issue: 10, price: price} = c1
		%Comics{issue: 10, price: 9.95, title: "Asterix"}

		iex> price
		9.95

See?  So simple.  And pattern matching is awesome.  But where, you might ask, does the Objective-C part come in?  Watch this:

		defmodule Comics do
		        @derive Access
		        defstruct title: "", issue: 1, price: 3.99
		end

Wait till I show you what that @derive doo-hicky does!

		$ iex comics.exs

		iex> c1 = %Comics{title: "Asterix", issue: 10, price: 9.95}
		%Comics{issue: 10, price: 9.95, title: "Asterix"}

		iex> c1[:title]
		"Asterix"

Yes!  Elixir has Square Brackets!  While Swift moves away from them, you still have a home in Elixir!  Welcome!

(And the associative arrays -- we call 'em "maps" -- do square brackets, too, but that's a lesson for another time...)
