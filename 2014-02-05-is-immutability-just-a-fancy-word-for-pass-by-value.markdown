---
layout: post
title: "Is Immutability Just a Fancy Word for Pass-By Value?"
date: 2014-02-05 22:18:51 -0500
comments: true
categories: fp
---
I'm new to functional programming.  I had an Artificial Intelligence course in college where we did a little Scheme.  Our final project was to program something related to the board game "Clue." As I recall, I did a relatively simple program that made guesses as to the killer's identity based on the cards in the players hand. Something like that.

That's been it, really.

But after plunging head first into the world of FP recently, something finally dawned on me:

Isn't "Immutability" just enforced "Pass By Value" on the part of the programming language?  Everything you send out to a function is a copy of the values, not the values themselves.  You don't pass pointers to variables, but rather a copy of the values represented by the variable, right? When the value is returned, "immutability" just states that you can't set the new value equal to the old value's variable host name, right?