---
layout: post
title: "Core Elixir: List.delete/2 and List.delete_all/2"
date: 2015-08-27 22:25:36 -0400
comments: true
categories: elixir coreelixir CoreElixir Elixir core delete Enum filter map reduce
---

{% img http://variousandsundry.com/cs/images/elixir_delete_all.jpg %}

[Last week, we looked at List.delete_at](http://variousandsundry.com/cs/blog/2015/08/20/core-elixir-list-dot-delete-at-slash-2/), which deletes a value at a specific position of a given list.

What if you know the _value_ you want to delete, though, but not the position?  Elixir has `List.delete/2` for that!

There is one catch, however.  If your list contains the value you're looking to delete in more than one place, it will only delete the first one. The rest will remain.

How does this function work?  Why doesn't it delete more than one value?  What kind of daring recursion knows to short circuit itself so strongly?  

Beats me. The function is an Erlang wrapper:

```elixir
  def delete(list, item) do
    :lists.delete(item, list)
  end
```

Digging into Erlang source code is beyond the scope for Core Elixir.

It does, however, perfectly follow one of the Elixir language's goals mentioned [at the top of the List library's documentation](http://elixir-lang.org/docs/v1.0/elixir/List.html):

 > A decision was taken to delegate most functions to Erlang’s standard library but follow Elixir’s convention of receiving the target (in this case, a list) as the first argument.


## Delete Them.  ALL of Them.

As a programming exercise, let's create a `List.delete_all/2` that won't stop after the first example of the value is found.

We'll try recursion with a dash of pattern matching:

```elixir
  def delete_all(list, value) do
    delete_all(list, value, []) |> Enum.reverse
  end

  defp delete_all([h|[]], value, end_list) when h == value do
    end_list
  end

  defp delete_all([h|[]], _value, end_list) do
    [h|end_list]
  end

  defp delete_all([h|t], value, end_list) when h == value do
    delete_all(t, value, end_list)
  end

  defp delete_all([h|t], value, end_list) do
    delete_all(t, value, [h|end_list])
  end
```

You have `List.delete_all/2` now that takes a list and the value you're looking to eliminate from that list. In turn, that calls the private function `List.delete_all/3`, where all the recursive fun begins. It goes through the list left to right, checking each member of the list to see if it matches the value you're trying to eliminate.  If it's a match, it skips over it and calls itself with the tail of the list you started with, and without adding any values to the new list you're constructing.

When you get to the final value, it checks for a match in the guard clause, and then returns the appropriate list.

The end result, naturally, is backwards, so the main function handles the reversing of the list to make it look right again.

Like this:

```elixir
iex> li = [0,1,0,0,1,0]
iex> List.delete_all(li, 0)
[1, 1]
iex> List.delete_all(li, 1)
[0, 0, 0, 0]

iex> li = [1,2,3,4,5]
iex> List.delete_all(li, 3)
[1, 2, 4, 5]
iex> List.delete_all(li, 5)
[1, 2, 3, 4]
iex> List.delete_all(li, 0)
[1, 2, 3, 4, 5]
```


## Take Two

Then I stopped to think, and remembered that functional programmers love three things:

 * __Map__ performs the same function to every item in a list. _(New List: Same Length)_
 * __Filter__ eliminates values from a list that don't match up to a function. _(New List: Smaller or Same Length)_
 * __Reduce__ takes a list and returns a single value derived from the items in the list.  _(Accumulator Value)_

This is a gross simplification, but it works for me.

When I look at what I coded above, it's basically a `map` function.  I'm going over a list and applying a function to each item, one by one, then returning a list.  But that's not the best abstraction for this problem.  

Map, by its very nature, is meant to apply a function to every item in a list and return that entire list, transformed.  My code basically mapped over a list and created a new list.  Kinda close, but not really it.

The problem I'm solving for here is an obvious __filter__.  We need to filter out the values from the list that we don't want:

```elixir 
  def delete_all(list, value) do
    Enum.filter(list, fn(x) -> x != value end)
  end
```

These three lines of code do the same thing as the other monstrosity I programmed earlier.  It filters the list, returning only the values for which the function is true -- that the item in the list isn't the same as the value we're looking to get rid of.  So it only returns values from the original list that don't equal the value you're looking to, well, "filter out." It's right there in pretty plain English.  And you can save 12 lines of code.

Map. Filter. Reduce.

Such helpful concepts.

Functional programmers love it so much, they even combine them.  [Elixir's Enum library](http://elixir-lang.org/docs/v1.0/elixir/Enum.html#flat_map/2) also contains `map_reduce/3` and 'filter_map/3`.  There's even something called `flat_map_reduce/3`.

## Filter It, Just a Little Bit (More)

But `filter` isn't as deep as we could go. Here's the `Enum.filter` source code:

```elixir
def filter(collection, fun) when is_list(collection) do
  for item <- collection, fun.(item), do: item
end

def filter(collection, fun) do
  Enumerable.reduce(collection, {:cont, []}, R.filter(fun))
  |> elem(1) |> :lists.reverse
end
```

We're dealing with a list here, so we get to use the much more simple first instance you see there.  That's a list comprehension that acts as a filter.  It goes through each item in the collection, runs it through a function to make sure it returns a truthful value, and then does something with it. In this case, it returns the value, un-altered, so long as the function turns out to be true when run against that item.

The second filter version works when your collection it not a list.  We're not going to worry about that one now.  We'll get back to `reduce` in a little bit, though.

What if we rewrote our `delete_all` function directly with the list comprehension code that `filter` uses?

```elixir
def delete_all_filter(collection, value) when is_list(collection) do
  fun = fn(x) -> x != value end
  for item <- collection, fun.(item), do: item
end
```

Double check that it works:

```elixir
iex> li = [1,2,3,4,3,4,5]
iex> List.delete_all(li, 4)
[1, 2, 3, 3, 5]
iex> List.delete_all(li, 3)
[1, 2, 4, 4, 5]
```

Yup, that'll do it.

I doubt this speeds anything up, but it does feel "closer to the metal."

If you were really feeling adventurous, though, you'd rewrite it as a straight `reduce`. 

...

Oh, we've gone this deep, already. Why not?  


## The Reduce Road

[Joe Kain has a great reduce write-up](http://learningelixir.joekain.com/reduce-patterns-in-elixir/) that I can't recommend enough. It's the best explanation of `reduce` I've seen so far, and I used it as inspiration for writing my function here:

```elixir
  def delete_all(collection, value) do
    Enum.reduce( collection, [], fn(x, acc) ->
      case x != value do
        true  -> [x | acc]   # Not the value, add it to the list
        false -> acc         # Matches the value, so don't add it
      end
    end)  |> Enum.reverse
  end
```

Note it still has the same user interface.  It's still a list and a value for the two parameters and that's it. The `reduce` function takes care of the rest.  The only complication here is the `case` statement I had to throw in. If there's a way around it without adding more functions, please drop me a line or an issue/pull request. I'd love to simplify this further, so long as it's readable.

The way this works is by using basically the same function we used above, but instead of just returning one value at a time, the reduce statement grows its own list as the list is traversed.  It does, however, return the list backwards, so we pipe the results to `Enum.reverse` to straighten it back out.  Now the input __and__ the output of this function are identical to the other options discussed above.

In the end, I think the `filter` solution is the cleanest and clearest to me. It certainly beats my original stab at the problem.  Maybe that's the lesson we learned today?  The further down you dig, the more options you have and the better your final function could be. Also, you might learn some stuff.

Sure, I'll go with that.

_If you have any comments, questions, complaints, criticisms, or corrections, catch me on Twitter, [@AugieDB](https://twitter.com/augiedb). Or [make a pull request on Github](https://github.com/augiedb/VariousAndSundryCS)!  That Twitter handle and Github ID is the same as my GMail account, if you want to deal with it more quietly. I want these articles to be factually correct and will update them as necessary._

_(12)_
