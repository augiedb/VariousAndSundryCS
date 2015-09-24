---
layout: post
title: "Core Elixir: List.foldl/3 and List.foldr/3"
date: 2015-09-22 22:52:05 -0400
comments: true
categories: elixir Elixir coreelixir foldrl foldr reduce enum core
---
 {% img http://variousandsundry.com/cs/images/drop_foldl.jpg %}

Before Elixir, I never dealt with folding functions.  Now, I have a foldl and a foldr function to wrap my head around.  It seemed so simple at first, but I had to bend my mind around this one for a couple of reasons.  Let's try to walk through these two functions and see what we can learn.

## Digging Not Too Deep

Before we go too far with this, let's jump to the end: `List.foldl` and `List.foldr` are wrappers for Erlang functions `:lists.foldl` and `:lists.foldr`.  Elixir rearranges the parameter order and that's it.

I mention this now because the Erlang documentation helps out with explaining these functions shortly.


## What Are We Folding Here?

`List.foldl` takes three arguments: a list, an accumulator, and a function.  The accumulator doubles as your starting point.

A fold is basically a reduction.  It takes a list, processes every item of it through a function, and returns a single value.

In fact, the Elixir `Enum.reduce/3` function is also a wrapper for Erlang's `:lists.foldl/3` function.

`List.foldl/3` applies the function to each element in the list, from left to right.  Let me repeat it so it doesn't get too confusing: _foldl runs from left to right._

It's a bit confusing in the Elixir documention, which says that the foldl function

 > Folds (reduces) the given list to the left with a function. 
 
To my mind, that would mean it runs right to left.  The Erlang documentation defines `foldr` as being "Like `foldl/3`, but the list is traversed from right to left."

I think the Erlang docs use the preposition better.

I've read other explanations of the two folds that rely on the "from" preposition to explain it better: `foldl` works _from_ the left, while `foldr` works its way _from_ the right.

## Does It Matter?

A lot of the times, it won't matter whether you go from the left or from the right.  If you're just summing up or even multiplying together the values in the list, for example, you'll get the same answer either way.  You may remember the "commutative property" from your childhood. That applies here. 

On the other hand, when order counts, there's a big difference between left and right.  Take subtraction, for one easy example that's also used in the Elixir docs:  

 * 1-2-3-4 is -8.  
 * 4-3-2-1 is a mere -2.

Fold statements are a little more complex than that, though, as they can use the accumulator value to start at a different place, or even in the calculations along the way.  Once you try to do something like `f(x) -> x - acc`, you have a new kettle of fish.

Also keep in mind that the accumulator is mandatory in your fold statements in Erlang and, thus, Elixir.  You don't need to use it, but you must include it.

Start the accumulator at 0 and let's `foldl` the list of the first four numbers and see where we go:

```
x  - acc = new_acc
------------------
1  - 0   = 1
2  - 1   = 1
3  - 1   = 2
4  - 2   = 2
```

Reverse that and let's subtract from the right with `foldr`:

```
x - acc = new_acc
-----------------
4 - 0   = 4
3 - 4   = -1
2 - -1  = 3
1 - 3   = -2
```

At each step of the way, `reduce` keeps track of which item on the list it's at, and what the value of the accumulator is.

The `reduce` function itself can be a complicated piece of machinery, and it's one that powers much of `Enum`.  Someday, I hope to get around to figuring out how it works in graphic detail.


## When in doubt, foldl

Why?  [It's tail recursive.](http://www.erlang.org/doc/man/lists.html#foldl-3)  This makes sense.  You're iterating over a list.  How many times have we seen now that lists work best when processed from left to right?   I suspect it has something to do with that.  And if it doesn't -- well, it's an easy way to remember it, at least.

For more detail on this, check out [this HaskellWiki article.](https://wiki.haskell.org/Foldr_Foldl_Foldl') 


## Heavier Reading

[This whitepaper](http://www.cs.nott.ac.uk/~pszgmh/fold.pdf) was a pretty interesting-looking appreciation of using a fold over recursion:

 > ...we show that even though the pattern of recursion encapsulated by fold is simple, in a language with tuples and functions as first-class values the fold operator has greater _expressive power_ than might first be expected.

Not that I understand a word of it.

_If you have any comments, questions, complaints, criticisms, or corrections, catch me on Twitter, [@AugieDB](https://twitter.com/augiedb). Or [make a pull request on Github](https://github.com/augiedb/VariousAndSundryCS)!  That Twitter handle and Github ID is the same as my GMail account, if you want to deal with it more quietly. I want these articles to be factually correct and will update them as necessary._

_(14)_
