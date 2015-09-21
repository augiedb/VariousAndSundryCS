---
layout: post
title: "Core Elixir Follow-Up: List.delete_all"
date: 2015-09-07 09:24:55 -0400
comments: true
categories: coreelixir elixir delete list
---

Three updates from [the List.delete_all post](http://variousandsundry.com/cs/blog/2015/08/27/core-elixir-list-dot-delete-slash-2-and-list-dot-delete-all-slash-2/) a couple weeks back:

## Naming is Hard

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/augiedb">@augiedb</a> <a href="https://twitter.com/elixirfountain">@elixirfountain</a> def delete_all(list) when is_list(list) do [ ] end # There, I implemented it for you :-)</p>&mdash; __red__ (@noidd) <a href="https://twitter.com/noidd/status/637424696447889408">August 29, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Yeah, that's a fair point.  A better name for the function might be `List.delete_all_of`, maybe?  ("Delete all of the 5s from this list.")  It's tough to be descriptive and terse at the same time. Nobody wants to type `List.delete_all_values_from_this_list_that_equal_this`.  We're not Objective-C with XCode to autocomplete all of that for us.  

I poke fun of Objective-C sometimes, but I admit that it's helped me to make lengthier and more descriptive variable names that I've appreciated more than once six months later...  I do, however, use Ruby's underscore syntax over CamelCase to make those variable and function names.  In Perl.

I'm a heretic.


## Refactoring

[I had a pull request from Daniel Garcia](https://github.com/augiedb/VariousAndSundryCS/pull/3), who made a smart observation about my original `List.delete_all` code.  Here's what I started with:

```elixir
  def delete_all(list, value) do
    delete_all(list, value, [] ) |> Enum.reverse
  end

  defp delete_all([ h | [] ], value, end_list) when h === value do
    end_list
  end

  defp delete_all([ h | [] ], _value, end_list) do
    [h|end_list]
  end

  defp delete_all([h|t], value, end_list) when h === value do
    delete_all(t, value, end_list)
  end

  defp delete_all([h|t], value, end_list) do
    delete_all(t, value, [h|end_list])
  end
```

I didn't think too much about it.  I got something to work and stopped there, because I knew I'd be working on the problem in different styles.  Still, there's an obvious pattern matching issue here that I should have noticed.  Take a look at these two functions, in particular:

```elixir
  defp delete_all([ h | [] ], value, end_list) when h === value do
  defp delete_all([ h | t  ], value, end_list) when h === value do
```

You can pattern match inside the arguments.  If you're looking for the case where `h` and `value` are the same thing, you don't need to put that in the guard clause.  You can match it inside the argument list: 

```elixir
  defp delete_all([ value | [] ], value, end_list) do
  defp delete_all([ value | t  ], value, end_list) do
```

Now, look at that and realize that in one major case, it's the same function.  When the tail is an empty list, the first version will be triggered instead of the second.  Can we combined those two into one?  Yes, if the base case (the one that returns the final results) is rewritten to look for just an empty list.

Those middle three functions can then be combined down into two:

```elixir
  def delete_all(list, value) do
    delete_all(list, value, []) |> Enum.reverse
  end

  defp delete_all([], value, end_list) do
    end_list
  end

  defp delete_all([value|t], value, end_list) do
    delete_all(t, value, end_list)
  end

  defp delete_all([h|t], value, end_list) do
    delete_all(t, value, [h|end_list])
  end
```

Your variants of the private function handle the cases where (1) the value is the same as the head, (2) the two are different, and (3) you have an empty list.  That's a lot more straight forward and obvious than my initial version was, where you had alternate versions of the same function depending on a guard statement that was redundant and the base case was one step removed from the end of the actual list. It just plain old makes more sense. 

## Reduce It Again

I wrote up a basic `reduce` function to do the same thing, admitting that I thought it might be done cleaner somehow.

In jumped [Kash Nouroozi](https://twitter.com/knrz_) with [a gist](https://gist.github.com/knrz) to change my function --

```elixir
def delete_all(collection, value) do
  Enum.reduce( collection, [ ], fn(x, acc) ->
    case x !== value do
      true  -> [x | acc]   # Not the value, add it to the list
      false -> acc         # Matches the value, so don't add it
    end
  end)  |> Enum.reverse
end
```

-- to something more elegant using `foldr` that doesn't even need the `reverse` at the end:

```elixir
def delete_all(collection, value) when is_list(collection) do
  List.foldr collection, [ ], fn
    x, acc when x == value ->
      acc
    x, acc ->
      [x|acc]
  end
end
```

I understand this new bit of code, but I'm afraid explaining it in graphic detail will have to wait for another day. I am working on a `foldl/foldr` edition of Core Elixir for some point in the future.  So stay tuned there.

_Thanks once again to Daniel and Kash for their contributions/pull requests/gists._ 

_If you have any comments, questions, complaints, criticisms, or corrections, catch me on Twitter, [@AugieDB](https://twitter.com/augiedb). Or [make a pull request on Github](https://github.com/augiedb/VariousAndSundryCS)!  That Twitter handle and Github ID is the same as my GMail account, if you want to deal with it more quietly. I want these articles to be factually correct and will update them as necessary._

