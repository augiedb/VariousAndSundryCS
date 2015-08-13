---
layout: post
title: "Core Elixir: Integer.is_even/1 and Integer.is_odd/1"
date: 2015-07-02 07:00:27 -0400
comments: true
categories: coreelixir Elixir elixir integer odd even
---

_Is this number odd or even?_

This should be simple, right? Oldest trick in the book: If the modulo (remainder) of the number divided by	 2 is 0 (has no remainder), then it's even. Otherwise, it's odd.

In fact, here, I'll give you an example with the Elixir `rem` command, which returns the remainder after dividing the two arguments you send it:

```elixir
iex> rem(10, 2)
0
iex> rem(11,2)
1
```
 
The first one is even.  The second one is odd. Done and done. Still simple, right?

Elixir's `rem` is actually `Kernel.rem` and calls out to Erlang's `:erlang.rem` to do the actual work.

But that's not how it works in Elixir.  It's much fancier, and much lower-level.

# Back to Bits

Elixir reaches into its back pocket and pulls out _a macro_ on this problem.

I'm not going to get into what a macro is or how it works here.  [Chris McCord wrote a book about it.](http://amzn.to/1KMouER)  Since I haven't read it yet, it would be irresponsible of me to attempt to break that down any further.

But the trick it uses to determine whether an integer is odd or even is [the Bitwise AND operator](http://en.wikipedia.org/wiki/Bitwise_operation#AND).  I'll attempt to provide a lesson on that today, instead.  It's likely equally irresponsible of me to attempt this, but I have a Computer Science degree, so we should be good, right?
 
# B to the AND

The Bitwise AND operator converts a number to bits, i.e. it turns it into 1s and 0s.  This is ridiculously handy for determining when something is even or odd. Why?  That last bit will give it away.  If the low bit (the one on the far right side) is turned off (0), the number is even. If it's turned on (add 1), the number is odd.

Remember, that all the other bits before that last number are even numbers (2, 4, 8, 16, etc.)  Only that last one, 1,  is odd.  And if there's anything you might remember from second grade math, it's that even + even is even and even + odd is odd.  (Odd + odd is also even, but we don't have two odd bits to play with.) 

So you AND 1 (that is, "00000001") to the number.  0 AND anything gives you 0, so you effectively zero out all the bits except for maybe that last one.  

It comes down to that last bit:  __1__ AND 1 gives you 1. __0__ AND 1 gives you 0.  A final value of 1 is odd, [0 is even](https://en.wikipedia.org/wiki/Parity_of_zero).  

In other words :

{% img http://variousandsundry.com/cs/images/elilxir_band_reminder_web.jpg %}

So, `is_odd` returns true when that computation results in 1, and `is_even` returns true when that computation results in a 0.  That's exactly what the source code does:

```elixir
defmacro is_odd(n) do
  quote do: (unquote(n) &&& 1) == 1
end

defmacro is_even(n) do
  quote do: (unquote(n) &&& 1) == 0
end
```

(Even not knowing much about macros, that code is pretty readable.)

One last example of how Binary AND works:

{% img http://variousandsundry.com/cs/images/BinaryAND.jpg %}

Now you can feel closer to the metal and, like a smarter and more powerful programmer. You just played with the bits!

# The Erlang Connection

One last thing I should mention: The `&&&` operator runs Erlang's `:band` function ( __B__ itwise __AND__, get it?)  [See the source here.](https://github.com/elixir-lang/elixir/blob/v1.0.4/lib/elixir/lib/bitwise.ex#L69)

'&&&' is included in Elixir's `Bitwise` module, which must be included before you can use it in your modules, [as it is in Elixir's Integer library.](https://github.com/elixir-lang/elixir/blob/master/lib/elixir/lib/integer.ex#L6)  Otherwise, you get an error that tells you to do just that:

```elixir
iex> Bitwise.&&&(20, 1)
** (CompileError) iex:22: you must require Bitwise before invoking the macro Bitwise.&&&/2
    (elixir) src/elixir_dispatch.erl:97: :elixir_dispatch.dispatch_require/6
```

I only point this out because the error message is so helpful. As I delve deeper into the Elixir core, I see more and more of the most helpful and specific error messages I've ever seen in a computer language.   That's handy.

So, use the library and compute away:

```elixir
iex> use Bitwise
iex> Bitwise.&&&(20, 1)  # 20 is even, so should return 0
0
```

Note that the return value is the same as if you had done the old divide-by-two trick.  No remainder indicates it's even.   And if there is a 1 leftover:

```elixir
iex> Bitwise.&&&(19, 1) # 19 is odd, so should return 1
1
```

Boom! Odd. 
 
 
# Including and Requiring

I mentioned before that `is_odd` and `is_even` are macros, right? That means that, unlike the Bitwise operator, we can't just `use` the Integer module.  We need to `require` or `import` it.  That will ensure that the macros are created and ready to go at compilation time.  ([The Elixir Getting Started Guide explains this fully,](http://elixir-lang.org/getting-started/alias-require-and-import.html#require) and even uses `Integer` as its example.)

What's the difference between the two?  With `require`, you still need to include the module name in your commands:

```
iex> require Integer
iex> Integer.is_even(20)
true
```

With `import`,  you don't need to use the module name anymore:

```
iex> import Integer
iex> is_odd(20)
false
```

You also have the ability to only import the specific functions you need, or to include only all of the macros in the module.

```
iex> import Integer, only: :macros
iex> is_even(20)
true
```

And Bob's your uncle. 

_If you have any comments, questions, complaints, criticisms, or corrections, catch me on Twitter, [@AugieDB](https://twitter.com/augiedb).  That handle is the same as my GMail account, if you need to type more characters. I want these articles to be factually correct and will update them as necessary._

_(4)_
