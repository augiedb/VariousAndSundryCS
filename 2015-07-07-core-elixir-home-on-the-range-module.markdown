---
layout: post
title: "Core Elixir: Home on the Range Module"
date: 2015-07-07 14:51:43 -0400
comments: true
categories: coreelixir elixir Elixir core
---

{% img http://variousandsundry.com/cs/images/elixir-gagsrange2small.jpg %} 

[It seems so small.](http://elixir-lang.org/docs/stable/elixir/) Two functions and you're done.

Then you open up the source and see the protocols at work. That's when you realize that the thing you think of as a range is really a little more complicated than that.  A range is not a core Elixir data type. It's actually just a `Range` struct:

```elixir
def new(first, last) do
  %Range{first: first, last: last}
end
```
It's a _simple_ and _specific_ struct with all of two values, as you might suspect: the first one and the last one.  You get the two dots in the middle for free.

The module also has a test to see if it's a range:

```elixir
def range?( %Range{} ), do: true
def range?(_), do: false
```

It's almost _too_ obvious, isn't it?  If you pass in an item that pattern matches a `Range` struct, then you're a range. Otherwise, you're not.

Naming wise, it's a bit different than you might expect. With other data types, Elixir comes pre_loaded with _is_ functions:  Does there exist an `is_range` function? Let's try it:

```elixir
iex> r = 1..3
1..3
iex> is_range?(r)
** (RuntimeError) undefined function: is_range?/1
```

Nope.  Let's take a look. From an iex prompt, type in `is_` and hit tab for the auto-complete options:

```elixir
iex(6)> is_
is_atom/1         is_binary/1       is_bitstring/1    is_boolean/1
is_float/1        is_function/1     is_function/2     is_integer/1
is_list/1         is_map/1          is_nil/1          is_number/1
is_pid/1          is_port/1         is_reference/1    is_tuple/1
```

Range isn't on that list.  Nor, you may notice is there an `is_struct`.  You need the `range?` function mentioned above instead:

```elixir
iex> Range.range?(r)
true
```

I have to admit that that does look a little weird to me.  Could look stranger, though:

```elixir
iex> range = 1..3
iex> Range.range?(range)
true
```

This tutorial is showing great range, isn't it?



## The Protocols. Oh, the Protocols!

There are three protocols that `Range` maps to, and that's what takes up the second half of the module.

First, the type is enumerable, so it has to define its `reduce` function to be compatible with everything in the `Enum` library.  When we get to the `Enum` library in a future "Core Elixir" installment, you'll see that almost all of the `Enum` functions are using `reduce`.

It also implements `member?` to tell if a value comes inside the range.

```elixir
def member?(first .. last, value) do
  if first <= last do
    {:ok, first <= value and value <= last}
  else
    {:ok, last <= value and value <= first}
  end
end
```

That's fairly simple, too, though Perl6 has it beat:

```perl
$low < $value < $high
```

I'm sure it has to do with the way languages are parsed and Abstract Syntax Trees and all the rest, but why doesn't every language these days have that?  Why do we need a special function or two tests to prove that out?

My kingdom for a macro!

## Count Along with Range

Finally, the `Range` module has a `count` function, which you can likely guess the purpose of.

Here's the funny trick there: The `count` function uses a count function that's defined as part of the Range.Iterator protocol!  Yes, it's protocols all the way down!

`Range.Iterator` has `count` and `next` defined, for what that might be worth.  I won't bore you too much with those because I bet you're smart enough to figure out how they work.

Finally, you need to have some way of inspecting the range so, of course, you implement the `Inspect` protocol!

```elixir
defimpl Inspect, for: Range do    
  import Inspect.Algebra

  def inspect(first .. last, opts) do
    concat [ to_doc(first, opts), "..", to_doc(last, opts) ]
  end
end
```

Some background: `to_doc` is in [the `Algebra` module](https://github.com/elixir-lang/elixir/blob/a60b603b7b6bbe2f6e49ddcf73dcae254f3e2717/lib/elixir/lib/inspect/algebra.ex#L196). I never would have guessed that, but thankfully the Elixir documentation has great search functionality.

The purpose of `to_doc` is to convert an Elixir structure to an "algebra document." In other words, it translates a number to a string so it can be printed out.  (This is a gross oversimplification, but it's good enough, I think.)

Long story, short: `inspect` makes sense, converting the struct to the first number, dot, dot, the last number.

```elixir
iex> inspect(r)
"1..3"
```

Hey, nothing in a programming language appears by magic, even something so simple of showing the value of a variable.  It all needs to be coded in somewhere.

## Your Range Power Tip of the Day

You can have descending ranges!  10..1 is just as valid as 1..10, though obviously your outcome will be wildly different for many functions.

Look through the Range code enough and you'll find places where that  has to be kept in mind as the program runs. The simplest example I can give you is the `Range.member?/2` function which, as you might imagine, checks a range to see if a specific value falls within it:

```elixir
def member?(first .. last, value) do
  if first <= last do
    {:ok, first <= value and value <= last}
  else
    {:ok, last <= value and value <= first}
  end
end
```


_If you have any comments, questions, complaints, criticisms, or corrections, catch me on Twitter, [@AugieDB](https://twitter.com/augiedb).  That handle is the same as my GMail account, if you need to type more characters. I want these articles to be factually correct and will update them as necessary._

_(5)_
