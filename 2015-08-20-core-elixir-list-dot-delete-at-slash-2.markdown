---
layout: post
title: "Core Elixir: List.delete_at/2"
date: 2015-08-20 23:00:36 -0400
comments: true
categories: core elixir list delete delete_at CoreElixir Elixir
---

{% img http://variousandsundry.com/cs/images/elixir_begging_color_lettering.jpg %}

The List module includes a bunch of functions that can't be done with an Enum function, which collections conform to.  If you're having a hard time deciding whether to handle something as a List or Collection, try the Collection first.  

But if you have a list and want to delete a single item on it, you've come to the right place!

## List.delete_at/2

The first thing you need to remember is that Elixir is immutable.  The return value from this function will be a __new__ list.  You can rebind it to the original list's name, but it will not be directly affecting the data you started with.  In reality, you're not so much deleting an item out of the list as you are generating a new list that happens to not contain this value.

I do not include this at the top because I made that mistake. Oh, no. I'm a blogger and, as with Wikipedia, an Unimpeachable Voice of Authority/Wisdom. I'm telling you this for a friend.  Or something.

Anyway --

`delete_at` takes two arguments: the list and an index, which is the position of the value you want to delete.  This position is __zero-based__.  Again, I remind you of this _for a friend_.

```elixir
iex> a = [1,2,3,4,5]
iex> List.delete_at(a, 3)
[1, 2, 3, 5]
```

Remember what I said before about immutability?

```elixir
iex> a
[1, 2, 3, 4, 5]
```

The value of the `a` list didn't change at all.  You can rebind, though:

```elixir
iex> a = List.delete_at(a, 3)
[1, 2, 3, 5]
iex> a
[1, 2, 3, 5]
```

This function will also accept a negative index, which will count from the end of the list:

```elixir
iex> List.delete_at(a, -2)
[1, 2, 5]
```

When you're counting backwards, you count 1-based and not zero-based.  Not that I'd accuse you of ever trying something silly like using a -0 index.  Oh, no.  That's for me to try _for_ you-- er, your friend:

```elixir
iex> a = [1,2,3,4,5]
iex> List.delete_at(a, -0)
[2, 3, 4, 5]
```
Remember, the index is 0-based and 0 is the same as -0 and is a valid number. If you try to delete -0, you're just deleting 0.

I do these things so you don't have to.

If you give a number that makes no sense, er, is _out of bounds_, the original list is returned:

```elixir
iex> a = [1,2,3,4,5]
iex> List.delete_at(a, 27)
[1, 2, 3, 4, 5]
```

There's nothing in the 27th (er, _28th_) position to delete, so Elixir doesn't do anything.

When you offer the function a negative number to count backwards with, it's all a trick. The source code won't be counting back, it'll do something else. Let's look at the source now so we can get that point.


## The Source

`delete_at` is a gatekeeper function, meant to make things a little easier for the programmer.  It exists to convert the negative index into a positive value first before passing everything along to `do_delete_at`, which is a private function that takes two values: the list and the index.

Let's start with a positive number example, and look more closely at `do_delete_at`.

It first takes care of the simplest solutions.  If the list is empty, it returns an empty list:

```elixir
  defp do_delete_at([], _index) do
    []
  end
```

Duh.

If you only want to delete the 0th element (which is the first value, remember, as well as the -0th), return just the tail:

```elixir
  defp do_delete_at([_|t], 0) do
    t
  end
```

With those cases out of the way, now we can get to the fun recursive stuff.  This one bent my mind a little, until I sketched it out.  Here's a good tip for figuring out recursive functions: Start at the base case and increment your way up.  In this case, I started with an index of 0 already: It returns the tail.

Let's look at the code all together, and then we'll walk through it with an index of 1:

```elixir
defp do_delete_at([_|t], 0) do
  t
end 

defp do_delete_at(list, index) when index < 0 do
  list
end

defp do_delete_at([h|t], index) do
  [h | do_delete_at(t, index-1)]
end
```

The third pattern would match `(list, 1)`, returning the head value followed by the results of putting `(tail, 0)` through the same function.  You pass in 0 (decrementing the index) the second time since the list is now one item shorter at the front. 

We've already seen what `(tail, 0)` is going to return -- the tail of the list you just passed in.  Thus, the head is thrown out.

So, this function returns the head of the original list, tosses out the next value, then returns everything else: You just deleted the second item on a list, effectively.

Once you have that in your mind, the rest of them are easy.  2, 3, 4, etc. just get to the point where you cut off the head and return the tail plus all the previous heads.

As a nifty bonus, you don't need to reverse the list at the end.  The process maintains the order as it goes along. 



## Counting Backwards

A negative index does something interesting.  There's no code for explicitly counting backwards.  `do_delete` instead calculates the forward-counting position in the list to delete.  It adds the length of the list (hello, `List.length/1`) to the negative number you provided and deletes that item.  

For example: If your list is the numbers 1 through 5 and you want to delete the -2nd item in the list (4), `do_delete` will add list length 5 to -2 and delete the element at position 3, which is the fourth item on the list.  

Voila!
  
That was "simple."


## Wait, What About That Middle Function?

_Update: 8/23/2015 Answering [a Reddit question](http://variousandsundry.com/cs/blog/2015/08/20/core-elixir-list-dot-delete-at-slash-2/) here, which I skipped over in the initial explanation._

What, this one in `List.do_delete_at`?

```elixir
defp do_delete_at(list, index) when index < 0 do
  list
end
```

That's there in case the negative value the user submits has an absolute value greater than the length of the list.  if the user does something silly like that, Elixir will just pass back the original list, much the same way it does when the user provides a positive index greater than the length of the list.

```elixir
iex> a = [1,2,3,4,5]
iex> List.delete_at(a, -100)
[1, 2, 3, 4, 5]
```

The gateway function (`List.delete_at/2`) doesn't test for a negative value's actual value. That happens inside the private functions' (`List.do_delete_at/2`) pattern matching.  This is what the gateway looks like:

```elixir
  def delete_at(list, index) do
    if index < 0 do
      do_delete_at(list, length(list) + index)
    else
      do_delete_at(list, index)
    end
  end
```

You'll see that while it checks to see if the index value being passed in is negative, it doesn't check to make sure that it isn't TOO negative. That happens in the next step.


## Coming Up Next Week

We'll take a look at a sister function of `List.delete_at`, how it differs, how we might improve it, create a new function, refactor it two ways, and then take a long nap.

_If you have any comments, questions, complaints, criticisms, or corrections, catch me on Twitter, [@AugieDB](https://twitter.com/augiedb). Or [make a pull request on Github](https://github.com/augiedb/VariousAndSundryCS)!  That Twitter handle and Github ID is the same as my GMail account, if you want to deal with it more quietly. I want these articles to be factually correct and will update them as necessary._

_(11)_

