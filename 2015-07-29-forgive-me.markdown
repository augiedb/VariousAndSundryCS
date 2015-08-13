---
layout: post
title: "Forgive Me"
date: 2015-07-29 00:01:35 -0400
comments: true
categories: elixir Elixir
---

I have failed the religion of functional programming this week.

I attempted to loop over a list to populate a map using `Enum.map`.

When that didn't work, I attempted a list comprehension.

When that didn't work, I blamed scoping issues with closures.

And then, just when I was ready to dive off a cliff, I realized the error of my ways.

I used **recursion**.

It solved all my problems, including one nasty function where I used five temporary variables just to do a proper match.  (Honest, I always planned to refactor that at some point...)

Confession is good for the soul, so I seek absolution in the most public of ways: Blogging.

(I already tweeted about it earlier tonight, so where else did I have to go?)

I have begun a six step process to grieve for these unearthly sins.  I think the sixth step is to publish the code as a gist or something.  A before and after.  We'll see if my ego grants me that privilege by the end of the week...

Kids, use recursion.  You can thank me later.
 
