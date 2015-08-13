---
layout: post
title: "Grasping Structs"
date: 2014-07-13 21:59:18 -0400
comments: true
categories: elixir
---
A blind spot in my Elixir programming experience to date has been structs.  I read a lot about them on-line, but didn't really get why I'd want to use them or how they're any better.  Coincidentally, I started to reread [Dave Thomas' "Programming Elixir"](http://pragprog.com/book/elixir/programming-elixir) book, which I hadn't done in about five or six major revisions. A lot has changed in it.  He's written entirely new sections, replaced examples, and a whole lot more.

I happened to trip across the struct section and it finally hit me why they're cool and useful.  They're like class-specific maps that come with their own names and an easy initial syntax.  They are maps that are their own type.  It's pretty cool.

That led me to do what I always do when I learn something new with Elixir: Rewrite my deck of cards routine.

Yes, that old thing.

Again. 

I'm as sick of it as you are at this point, but it's handy for learning lots of basic syntax and Elixir theory, so here I go again.

I'm not sure I have much to explain here, so I'll just throw out the code here and give a couple annotations afterwards.


#card.ex

		defmodule Card do

		    defstruct suit: "", rank: "", value: ""

		    def describe(c), do: "#{c.rank} of #{c.suit} (#{c.points})"

		    def init_(rank) when rank > 1 and rank < 11, do: rank
		    def init_points('Ace'), do: 1
		    def init_points(_), do: 10

		end

I recognized something incredibly stupid in my original "Card" code. On the "init_points('Ace')" line, I was using a guard clause instead of just pattern matching on the argument.  Silly me.

Also, I'm all for writing clear and expressive and verbose syntax, but something drives me nuts about code that names variables the same as the class name.  That's why the describe function's argument is a "c" instead of "Card."

#deck.ex

		defmodule Deck do

		  def new do
		    for rank <- ['Ace', 2, 3, 4, 5, 6, 7, 8, 9, 10, 'Jack', 'Queen', 'King'],
		        suit <- ['Hearts', 'Diamonds', 'Spades', 'Clubs'] 
		    do
		      %Card{rank: rank, suit: suit, value: Card.init_points(rank)}
		    end
		  end

		  def shuffle(deck) do
		    deck |> Enum.shuffle
		  end

		  def count_suit(deck, suit) do
		    Enum.count(deck, fn(x) -> x.suit == suit end)
		  end

		  def count_rank(deck, rank) do
		    Enum.count(deck, fn(x) -> x.rank == rank end)
		  end

		end

Remember what I said above and then see where I use "rank: rank, suit: suit" in this code.  I am not a consistent person, sometimes.  I think it drives me the most nuts in pattern matching, though, where at first glance I can't be sure if, for example, "rank" were referring to something of a rank type, or a rank, itself.  Does that make sense?  It's probably just a style thing I need to get used to and then get over.

Those last two functions are there for future reference and for testing purposes.  When my program needs to figure out how many hearts are left in the deck, or how many jacks there still are, I can use that code.

One last word of warning: I once started writing a deck of cards Elixir module.  I started it as an experiment in using different data types and syntax construction. Then I realized I was overthinking things to a ridiculous degree, and abandoned it.  One of these days, I want to write up why we can something overthink modelling a real world scenario in code, using that dead program as an example.