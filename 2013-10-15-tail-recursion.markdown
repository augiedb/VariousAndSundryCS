---
layout: post
title: "Tail Recursion"
date: 2013-10-15 22:38
comments: true
categories: elixir, recursion
---
In learning a bit of Elixir last night, I had to pound into my head what "tail recursion" meant.  What I learned is that it saves (potentially) *lots* of memory by not storing all those frames during recursion, but that it does so at the expense of being a cool-looking piece of recursion.  

To me, the magic of recursion has always been the way the work is done bottom-up at the end as the recursion works its way back to the surface.  

Tail recursion winds up being more loop-like. It also usually winds up taking three parameters instead of one, so you lose some of elegance.

It took reading [this Stack Overflow question and set of answers](http://stackoverflow.com/questions/33923/what-is-tail-recursion) to figure it all out.  [Pay particular attention to this answer.](http://stackoverflow.com/a/34540/1546910)

