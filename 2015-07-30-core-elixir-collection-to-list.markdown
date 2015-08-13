---
layout: post
title: "Core Elixir: Collection to List"
date: 2015-07-30 23:36:57 -0400
comments: true
categories: elixir Coreelixir coreelixir Elixir collections lists
---

{% img http://variousandsundry.com/cs/images/elixir_pointing_final_web.jpg %}

First, we need to define some terms:  Elixir __Collections__ include things like HashDicts, Tuples, and Lists.  A __List__ is a very specific type of collection: It's a singly linked list, basically.  It has an _order_, which the other collections don't have.
 
If you want to do anything with a list, it's expensive. You need to traverse the whole list before you can do much with it. You can't get to the _nth_ member of the list without going through the first _n-1_ members to get there. There's no direct references to individual members of the list. No indices. No extra pointers.

The `Enum` module works with collections.  It takes the place of the various types of loops you might write in other languages.

Lists, in particular, have their own module, cleverly named `List`, that handles functions that make sense only to lists and not collections as a whole.  You can flatten a list there, find an item in a specific position, or fold a list to the left or right, for four examples.

So, to sum it up:

 * Lists are a special form of Collections.
 * `Enum` deals with Collections.
 * `List` deals with list-specific things an Enum isn't appropriate for.

With that out of the way:

## Conversion Therapy
 
Can we convert a collection to a list?  Of course we can!  We use `Enum.to_list/1`.

The crazy thing is __how__ that function does it.  

Remember that with Elixir it's easy to separate the head (first value) from the tail (everything afterwards) of a list.  It's _not_ easy to do the reverse and break the last value off the rest of the list.  (The [`List.last` function](https://github.com/elixir-lang/elixir/blob/v1.0.4/lib/elixir/lib/list.ex#L154) does this by traversing its way all the way to the end in tail-recursive style.)  You need to always move from the front to the back of a list; performance can be rather dismal, particularly as your list gets bigger.

There's no direct way of converting a collection into a list.  You can't wave a magic wand and have it happen.  Instead, you trick Elixir into looping over every value in the collection and adding those values to a list.  As it happens, that's a side effect for one of the `Enum` functions!  How convenient.

## Thinking Backwards

The Elixir `Enum.reverse` function takes in a collection and reverses it, making it a list in the process.  [It uses `reduce` and everything.](https://github.com/elixir-lang/elixir/blob/v1.0.4/lib/elixir/lib/enum.ex#L1366) Look:

```elixir
  def reverse(collection, tail) do
    reduce(collection, to_list(tail), fn(entry, acc) ->
      [entry|acc]
    end)
  end
```

That `reduce` statement goes item by item in the collection and keeps putting the next item at the head of a new list.  See that line perfectly in the middle of the code?  `[entry|acc]` is forming that list.   `acc` is the list so far (the accumulator), and `entry` is the head of the remaining collection that gets stapled into front of the list.

When all is said and done, you have a list of elements that came out of a collection.

The new list it creates, however, is in the reverse of the order the collection started with, since you're always placing the next element ahead of all the others so far. 

For example, we'll take a new Map (which is a collection), give it some values, and see what happens when we `Enum.reverse` it:

```elixir
iex> m = Map.new()
%{}
iex> m = Map.put_new(m, :a, 1)
%{a: 1}
iex> m = Map.put_new(m, :b, 2)
%{a: 1, b: 2}
iex> m = Enum.reverse(m)
[b: 2, a: 1]
```

_(Yes, I know the `Enum.into` trick, but for clarity's sake, I'm spelling this out long hand.  Maybe we'll talk about `Enum.into` in the future...)_

Look at that: You have a list now.  You can tell because it's surrounded in brackets and not a percent sign and curly braces.  Or, you can tell programmatically:

```elixir
iex> is_list(m)
true
```

Elixir's `List` module doesn't contain a reverse function.  That's because the `List` module only contains functions that wouldn't make sense as an `Enum` function.

Up to this point, you've done two things at the same time: Converted a collection to a list, and reversed the order.  That's more than you wanted to do, though.  You just wanted the conversion part, not the reversal part.

Since you just wanted to convert the collection to a list,  you need to reverse the list back into its original order.  Since it's a list now, it makes sense to use the list version of `reverse`.  On the off chance you skipped what I wrote two paragraphs ago: There is no Elixir `List` version of the reverse function.

We do the next best thing: we call directly out to Erlang's.  


{% img http://variousandsundry.com/cs/images/erlang_pointing_final_web.jpeg %}

That follows Elixir's standards. Per [the List module documentation](http://elixir-lang.org/docs/v1.0/elixir/List.html):

> A decision was taken to delegate most functions to Erlang’s standard library but follow Elixir’s convention of receiving the target (in this case, a list) as the first argument.

In any case, whether you do the second reversal with the Enum or `:lists` library, it will work, since we're dealing with a list at that point, either way:

```elixir
iex> Enum.reverse(%{a: 1, b: 2}) |> Enum.reverse
[a: 1, b: 2]
iex> Enum.reverse(%{a: 1, b: 2}) |> :lists.reverse
[a: 1, b: 2]
```

But the source code goes with the latter.

That's 800 words to describe what is a one-liner in Core Elixir:

```elixir
  def to_list(collection) do
    reverse(collection) |> :lists.reverse
  end
```

But we're not done yet!

## The Easier Way (Doesn't Work)

_05 August 2015: Major updates to change the example code in this section to be the same as the Maps-based example above.  A new section has been added right afterwards, as well._

Wait, isn't there a way to traipse across the collection one item at a time and _not_ reverse the subsequent list?

What if we took the `Enum.reverse` function and rewrote it?

Here again is what [the key part of it looks like](https://github.com/elixir-lang/elixir/blob/v1.0.4/lib/elixir/lib/enum.ex#L1366) today:

```elixir
  def reverse(collection, tail) do
    reduce(collection, to_list(tail), fn(entry, acc) ->
      [entry|acc]
    end)
  end
```

As an exercise, try flip-flopping the ```[entry|acc]``` to give you ```[acc|entry]```.  Don't you think that might stop the loop over the collection from returning results in reverse order?  

Sorta, but the results are not pretty:

```
[[[] | {:a, 1}] | {:b, 2}]
```

If you tease that apart, the first item in the list (the head) is a list of an empty list with a tail of {:a, 1}.

The `reverse/2` call is fronted by this `reverse/1` call:

```elixir
  def reverse(collection) do
    reverse(collection, [])
  end
```
The programmer just sends in a collection and doesn't worry about it. Let the library worry about the accumulator.  And, here, it's seeded as an empty list, [].

Since we started the recursion with [] as the list, that starts as the head with {:a, 1} as the tail.  Then, that whole construct becomes the new head, while the next value, {:b, 2} becomes the tail of a list where the head is an empty list and {:a, 1}.

If we extended the original map out a little bit, the new list looks even wonkier.  (I rewrote the reverse function in a new module I named after myself.  I'm not just an egomaniac, but I am very fast at typing my own name.)

```
iex> Augie.reverse(%{a: 1, b: 2, c: 3, d: 4, e: 5, f: 6}) 
[[[[[  [[] | {:a, 1}] | {:b, 2}] | {:c, 3}] | {:d, 4}] | {:e, 5}] | {:f, 6}]          
```

This is the head of that list:
```
[[[[[[] | {:a, 1}] | {:b, 2}] | {:c, 3}] | {:d, 4}] | {:e, 5}]
```

And the tail:
```
{:f, 6}
```

So I suppose you could do a weird kind of reverse recursion where you keep dealing with the tail and passing the head along. You'd process the list backwards.  And since that would go against every bit of conventional wisdom in programming, it's probably safe to ignore that.  It also doesn't bring us closer to converting a collection to a list without the second reverse.

This is how the new list is then constructed:

```elixir
[ [] | {:a, 1} ]
[ [ [] | {:a, 1}] | {:b, 2}]
[ [ [ [] | {:a, 1}] | {:b, 2}] | {:c, 3}]
[ [ [ [ [] | {:a, 1}] | {:b, 2}] | {:c, 3}] | {:d, 4} ]
etc. 
```

_I stopped there before you got dizzy from the brackets and pipes.  I've spent my whole career avoiding Lisp.  This is getting perilously too close._

Note the distinct lack of commas in there.  You can't flatten that list if you tried.  And, yes, I tried.  Because I'm thorough:

```elixir
iex> list = [[[[[[[[] | {:a, 1}] | {:b, 2}] | {:c, 3}] | {:d, 4}] | {:e, 5}] | {:f, 6}], {:g, 7}]
iex> List.flatten(list)
** (FunctionClauseError) no function clause matching in :lists.do_flatten/2
    (stdlib) lists.erl:625: :lists.do_flatten(9, '\n')
    (stdlib) lists.erl:626: :lists.do_flatten/2
```

__You break Erlang__ with that crazy request...  Congratulations.


## New on August 5, 2015	 - You CAN Do It!

I received an email from a reader, Roman, who made a smart suggestion to help fix this.  Instead of `[ acc | entry ]`, try `[ acc | [entry] ]`.  The system is expecting a list after the pipe "|", so give it one. 

The new `reverse` function looks like this:

```elixir
defmodule Augie do
	def reverse(collection, tail \\ []) do
		Enum.reduce(collection, Enum.to_list(tail), fn(entry, acc) ->
			[acc|[entry]]
		end
	end
end
```

Look at the big difference that gives us in results, before and after:

```elixir
iex> Augie.reverse(%{a: 1, b: 2, c: 3, d: 4, e: 5, f: 6}) # [ acc | entry ]
[[[[[[[] | {:a, 1}] | {:b, 2}] | {:c, 3}] | {:d, 4}] | {:e, 5}] | {:f, 6}]

iex> Augie.reverse(%{a: 1, b: 2, c: 3, d: 4, e: 5, f: 6})  # [ acc | [entry] ]
[[[[[[[], {:a, 1}], {:b, 2}], {:c, 3}], {:d, 4}], {:e, 5}], {:f, 6}]
```

All of those pipes for the head/tails separators have been replaced by glorious commas.  Now you can flatten it:

```elixir
iex> Augie.reverse(%{a: 1, b: 2, c: 3, d: 4, e: 5, f: 6}) |> List.flatten
[a: 1, b: 2, c: 3, d: 4, e: 5, f: 6]
```

Doesn't that look prettier now?   And you don't need to reverse it anymore, either!

So why _not_ go this way?  It's slower.

Just as a down and dirty test, I ran these two lines in `iex` to see how many milliseconds it would take to create a list with 100,000 entries and create a flat version of it.  I used the [Erlang :os.timestamp function](http://www.erlang.org/doc/man/os.html#timestamp-0) to grab the time before  and after each calculation.  Even with the extra step, the current `Enum.reverse` clearly wins:

```elixir
{_,_,c} = :os.timestamp; 1..100000 |> Augie.reverse |> List.flatten;  {_, _, c1} = :os.timestamp; IO.puts c1 - c;

{_,_,c} = :os.timestamp; 1..100000 |> Enum.reverse |> List.flatten |> :lists.reverse; {_, _, c1} = :os.timestamp; IO.puts c1 - c;
```

The results are never identical, but the range for the current `Enum.reverse` solution sits somewhere in the 20,000 - 23,000 microseconds range, while the `Augie.reverse` solution ranges between 24,000 and 30,000 microseconds.

Thanks again to Roman for pointing this out.  It's a good reminder to keep more proper lists...

## The Anti-Climax

This is not an essay that ends with a brilliant pull request to convert a collection into a list in one less step with moderate gains in speed and performance.

I don't have an answer to this.  Honestly, this isn't a problem that needs a solution.  That's not why I started writing this one.  This is about finding out how Elixir works behind the scenes.  What are the quirks of the language?  Where does it hand things off to Erlang? What would be useful for you to know as a programmer?

Sometimes, it's a winding path that goes in circles as we blindly grope for an answer.  Or an explanation.

It's that little kid's tendency to ask "Why?" constantly that drives this series.

Even when we hit the bottom without a clickbait twist to put in the headline.

It's just plain old Elixir.  And it's lots of fun.

Did you ever think the way to convert one data type to another is to use two functions that do completely unrelated things to the task at hand,  but do the same thing logistically in two different languages?  Crazy, right?

## Post Script: Did You Know?

`:lists.reverse` has two different versions in Erlang.  You have your pick between sending one argument or two.  What's the difference?  The first argument in both cases is the list you're looking to turn around.

When the arity is 2, though, the second argument is another list that will be added to the end of your reversed list, just in case you need that kind of thing:

```elixir
iex> :lists.reverse([1,2,3,4,5])
[5, 4, 3, 2, 1]
iex> :lists.reverse([1,2,3,4,5],[100,200,300])
[5, 4, 3, 2, 1, 100, 200, 300]
```

Note that the second list doesn't get reversed.

If your second argument isn't a list, it becomes the list's new tail:

```elixir
iex> :lists.reverse([1,2,3,4,5],1001)
[5, 4, 3, 2, 1 | 1001]
```

(Again, don't try to run `flatten` on that.  It doesn't work that way.  Weren't you paying attention 500 words ago?!?)

Elixir has the same thing. When you run `Enum.reverse` against  a collection, it actually calls `Enum.reverse/2`, with an empty list as the the tail.

But if you call `Enum.reverse/2` on purpose with some list to add to the end of a collection, then you're actually using an optimization in the language.  Elixir could just reverse the collection and then append the tail to it like this:

```elixir
Enum.concat(Enum.reverse(collection), tail)
```

Instead, Elixir pulls out Yet Another Reduce function:

```elixir
reduce(collection, to_list(tail), fn(entry, acc) ->
      [entry|acc]
    end)
```

Really, is there anything that `reduce` can't do?


## Summing it all up

If you feel the need to convert a collection to a list, just reverse it.  Twice.  Once in Elixir, once in Erlange.

For extra credit, pin a tail on it afterwards.

_If you have any comments, questions, complaints, criticisms, or corrections, catch me on Twitter, [@AugieDB](https://twitter.com/augiedb).  That handle is the same as my GMail account, if you need to type more characters. I want these articles to be factually correct and will update them as necessary._

_(8)_


