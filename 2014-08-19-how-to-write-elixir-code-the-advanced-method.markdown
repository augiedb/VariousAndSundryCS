---
layout: post
title: "How To Write Elixir Code: The Advanced Method"
date: 2014-08-19 01:55:47 -0400
comments: true
categories: elixir
---

_Just in case: The following is a work of loving parody.  Maybe satire?  Maybe just having fun.  Ya know I love ya guys and gals, right?_

##1. State Its Purpose

I want to write a program to generate even numbers.

(For the purposes of this tutorial, I'm choosing a ridiculously simple purpose, one so simple that it's already solved by the core of the language itself. Just work with me here.)


##2. Write a Test with ExUnit 

```ruby
    assert Number.is_even(0) == true
```

I looked it up on Wikipedia to be sure this was true. Then I tested it in ExUnit to make sure it passed.  Only THEN did I show you this test as if I was so confident it would work because I knew in advance that 0 is considered even.

If you have a problem with that, I invite you to look deep inside yourself and define "truthy" out loud with a straight face.

This is computer programming.  We're paid to make stuff up as we go along with utter confidence that Stack Overflow was right.


##3. Write the Code to Pass the Test

    defmodule Number do

      def is_even(0), do: true

    end

Computer programming has gone so high level that any fifth grader could do this stuff after a day in MineCraft...


##4. Repeat 2 and 3 As Often As Possible to Complete What Step 1 Asks for

Bonus points for using guard clauses or recursion.  I got one of those two in here.

```ruby
defmodule NumbersTest do
  use ExUnit.Case

  test "Even Numbers Check" do
    assert Number.is_even(0) == true
    assert Number.is_even(1) == false
    assert Number.is_even(2) == true
    assert Number.is_even(3) == false
    assert Number.is_even(4) == true
    assert Number.is_even(9) == false
    assert Number.is_even(200) == true
  end
end
```

Where's QuickCheck when I need it?  Or is that Concuerror I need?  I can never keep those two straight.  I do, however, recognize that ReBar wouldn't help here.  Yet.


```
    defmodule Numbers do

      def is_even(0), do: true

      def is_even(value) when rem(value, 2) > 0 do
        false
      end

      def is_even(_), do: true

    end
```

So far, so normal.


##5. Now, Rewrite it, With More Pipes

I picked far too simple an example for this. Trust me, though, if you want to hang with the cool kids, everything is a transformation, and every step of that transformation comes after a pipeline operator. Every program, once it's boiled down to its most basic structure, is this:

```ruby
Number
  |> Function
  |> Print
```

Remember that.


##6. Good, Now Take Out all the Parentheses You Can

```
    defmodule Number do

      def is_even 0, do: 1
      def is_even(value) when rem(value, 2) > 0 do
        value+1
      end
      def is_even(_), do: true

    end
```

It's usually harder to read, but the cool kids will like you more.


##7. Use OTP 

No matter what your program does, it'll be sweeter if 100 processes are doing it at the same time, waiting to receive your message and increment a counter somewhere. If you can break up the task to run across multiple servers at the same time, you'll also be crowned King of FP for the moment.

I changed the program to return the next even number.

```ruby
defmodule Even.Server do

  use GenServer

  def handle_call(:prime_or_not, _from, number) do
    {:reply, Number.is_even(number), Number.is_even(number) }
  end

  def next_even_number(number) do
    Even.Server.handle_call(:prime_or_not, number)
  end

end
```

##8. Post it to Hex.pm

Instructions yet to come. Get it in now, though, while all the good names aren't already taken.


##9. Brag on Twitter that you've reinvented the wheel, but it's functional!

<blockquote class="twitter-tweet" lang="en"><p>I just reinvented the wheel! This time, it’s functional! <a href="https://twitter.com/hashtag/MyElixirStatus?src=hash">#MyElixirStatus</a></p>&mdash; Augie De Blieck Jr. (@augiedb) <a href="https://twitter.com/augiedb/statuses/502170829699174400">August 20, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

##10. Go back to getting paid for making Ruby on Rails sites for corporate intranets.

I saw your fingers twitching there.  I know what you want to do. I know what you're thinking:

```ruby
rails new
```

##11. Convert it to Modern Elixir

That is, scrap your OTP port and convert it to the Phoenix framework.


##12. Extra Credit!

Rewrite the program using as many macros as possible....

Now go update your resume with "Elixir Ninja" and consider yourself "Full Stack"!