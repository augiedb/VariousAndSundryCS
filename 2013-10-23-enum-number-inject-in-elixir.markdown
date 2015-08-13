---
layout: post
publish: true
title: "Enum.Inject in Elixir"
date: 2013-10-25 11:00
comments: true
categories: elixir, enum
---

[The other day, I showed you Ruby's Enum#Inject method.](http://variousandsundry.com/cs/blog/2013/10/22/exploring-ruby-enumerable-number-inject/)  Then, I discovered that Perl had it built in through the ```List::Utils``` optional module.

Today, let's use this as a learning experience with [Elixir](http://elixir-lang.org).  I wrote the method up for Elixir like so:

```ruby
defmodule Example do

  def sumup(list) do
    sumup(list, 0)
  end

  defp sumup([], sum) do
    sum
  end

  defp sumup([heda | tail], sum) do
    sumup( tail , sum + head )
  end

end
```

I named the module ```Example``` and the function ```sumup```.

Just to be a little trickier, I set the function up to accept only the list from the caller.  Thus, ```sumup``` has an *arity* (number of parameters) of 1, so we refer to it as ```sumup/1```.

That initial ```sumup/1``` is just the public facing function.  Its sole purpose is to make it a little simpler on the user to calculate the total of all values in the array by not having to specify a starting total.  This program runs on the assumption that you're always starting from zero.  

It then calls on two private functions (```defp``` denotes that) to do the dirty work.

Specifically, we start with ```sumup/2``` where pattern matching kicks in for the first value.  If the list is empty, the first version kicks in and returns up the sum.  If the list is blank from the start, then it immediately returns the sum, which would be just passed in as 0.

Assuming there's a list (even if it's just one value), then we go to the second ```sumup/2``` function. Pattern matching kicks in automatically on the list, and separates the first value ("head") from the rest of the list ("tail").  It then recursively calls itself, passing the tail alongside the new sum (which now has added in the value of the head of the list the function started with).

Once you get to the last value of the list, "tail" is an empty list, "sum" is the total sum of the numbers in the list, and the first ```sumup/2``` kicks in and returns the sum you had asked for in the first place.

Let's see this in action.  First, I'll start Interactive Elixir (*iex*) and load up the file with the Example module in it:

```ruby
Augies-iMac:elixir augiedb$ iex
Erlang R16B01 (erts-5.10.2) [source-bdf5300] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]

Interactive Elixir (0.10.1) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> c("example.ex")
/Users/augiedb/ruby/elixir/example.ex:1: redefining module Example
[Example]
```

Obviously, this isn't the first time I've loaded up the Example module...

Let's start with the empty list:

```ruby
iex(2)> Example.sumup []
0
```

As we expected. Parenthesis are optional.  I like using parenthesis, generally, but for a quick and dirty example like this, I'm OK going without:

```ruby
iex(3)> Example.sumup([])
0
```

That does look a little weird.

So let's see what happens when we have a list with one value in it.

```ruby
iex(4)> Example.sumup [1]
1
```

That does the trick. Finally, for multiple values in a list:

```ruby
iex(5)> Example.sumup [1,2]
3
iex(6)> Example.sumup [1,2,55]
58
iex(7)> Example.sumup [1,2,55,1]
59
```

Elixir has [a built-in reduce function in its Enum module](http://elixir-lang.org/docs/stable/Enum.html#reduce/2), so you could always go with that:

```ruby
iex(8)> Enum.reduce([1, 2, 3, 4], 0, fn(x, acc) -> x + acc end)
10
```

For some reason, ```reduce/2``` isn't working for me, so I needed to go with ```reduce/3``` instead.  Odd, that, but hardly insurmountable.