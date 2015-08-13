---
layout: post
title: "Elixir Case Statements"
date: 2014-08-06 11:29:17 -0400
comments: true
categories: 
---
I had a stupid simple Elixir question the other day that none of the documentation I looked up had the answer for.

Then I felt dumb and realized I could test the idea myself far faster than it was taking me to look it up in books and [DDG](http://ddg.gg) it.

That idea is, can a `case` statement have more than one statement between pattern matches?  Every example I've ever seen has just one line.  But then I came across an Erlang program that I wanted to translate into Elixir that had two lines for each match.

Here, run this:

```ruby
IO.puts "CASE WITH ONE LINE:"
case 1 + 1 do
  11 -> IO.puts "Wrong"
  2  -> IO.puts "RIGHT"
end
```

And you get what you expect:

```
CASE WITH ONE LINE:
RIGHT
```

We knew that would be the case.  What happens with multiple lines, though?

```ruby
IO.puts ""
IO.puts "CASE WITH TWO LINES:"

case 1 + 1 do
  2 ->
     IO.puts "Line 1"
     IO.puts "Line 2"
  10 -> 
     "WRONG"
end
```

Yup, it works:

```
CASE WITH TWO LINES:
Line 1
Line 2
```

You don't have to add any extra characters or do/doend statements or curly braces or anything to cover that chunk of code.

So, rest easy.  The obvious solution works best here.

