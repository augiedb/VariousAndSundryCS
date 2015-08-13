---
layout: post
title: "Elixir Update: No More LCs"
date: 2014-05-28 13:29:05 -0400
comments: true
categories: elixir
---

Since I started programming in Elixir, I've been tweaking some routines to deal with playing cards.  Chief amongst them is the deck creation function:

	  defmodule Deck do

		  def create do 
		    # 52 cards, unless otherwise specified?
		    lc rank inlist ['Ace',2,3,4,5,6,7,8,9,10,'Jack','Queen','King'], 
		       suit inlist ['Hearts','Clubs','Diamonds','Spades'], 
		    do: Card.create( rank, suit, Card.init_points(rank) )
		    |> shuffle 
		  end 

		  [...]
		  
	  end

This routine uses the list comprehension syntax ('lc') to create a 52 card list.  Each card is created in Card.create.

I saw on the Elixir mailing list recently, though, that list comprehensions were being deprecated.  There was a new syntax floating about for that trick.  And so it came time to rewrite the deck creation function.  Now, it is this:

	  def create do 
	    # 52 cards, unless otherwise specified?
	    for rank <- ['Ace',2,3,4,5,6,7,8,9,10,'Jack','Queen','King'], 
	        suit <- ['Hearts','Clubs','Diamonds','Spades'], 
	    do: Card.create( rank, suit, Card.init_points(rank) )
	    |> shuffle 
	  end 

Minor difference, but an important one.  It's like doing nested ```for``` loops instead of a bit of magic syntax. The list comprehensions aren't long for this world, so it's useful to write them out now.

The cards, you may recall, are maps. Initially, they were records.  But then Elixir deprecated those, as well.  What, I wonder, is next for this poor little function?

