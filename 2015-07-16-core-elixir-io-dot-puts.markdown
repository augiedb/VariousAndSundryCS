---
layout: post
title: "Core Elixir: IO.Puts"
date: 2015-07-16 23:49:31 -0400
comments: true
categories: elixir Elixir coreelixir
---

{% img http://variousandsundry.com/cs/images/elixir-jurassic-world_tiny.jpg %}

_Stand back. We're going to wrestle some dangerous concepts to the ground, though no processes will actually appear..._

The string you're _IO.puts_ ing can be interpolated.  That's computer science speak for "the language will replace variables with their values inside the quotation marks."

Like Ruby, Elixir uses `#{}` for this:

```
iex> str = 'world'

iex> IO.puts("Hello, #{str}")   # Double quoted strings
Hello, world
:ok

iex> IO.puts('Hello, #{str}')   # Single quoted strings
Hello, world
:ok
```

In Elixir, the interpolation will happen whether you use single or double quotes.  You'll see I tried it both ways there. The difference in quotation styles is used only in dealing with Elixir strings versus Erlang strings, and we'll get to that at some point in the nebulous future once the Advil kicks in from getting that terminology right.

You'll notice that Elixir threw in a new line at the end of the string for us.  That's part of the service `IO.puts` offers.  In other languages, there's a different command for when you want to use or not use the newline.  In Perl, there's `print` and `say`. In Ruby there's `puts` and `print`. In both cases, the `print` leaves off the newline.

In Elixir, `IO.write` will not add the newline and print the string as is.

This can look awkward in the iex REPL:

```elixir
iex> IO.write('Hello, #{str}')
Hello, world:ok
```

Both `IO.puts` and `IO.write` allow you to send your output to different locations, such as standard error.  We'll cover that more in a future article on `IO.puts` that'll blow your mind!  (When it comes to managing expectations, I'm a failure.)

But, you may ask yourself if you're bored one day, how does interpolation work?


## How Does Interpolation Work?

I'm glad you asked. It's sorta like polymorphism. Yes, welcome to that rabbit hole...

The first trick is that Elixir sees `#{}` and rewrites it internally as `to_string` and then concatenates it with its surroundings.

_"Hello, #{str}"_ becomes _"'Hello, ' <> to_string(str)"_.

`to_string` is in the Kernel module (and is thus automatically imported) and adheres to the `Strings.Chars` protocol. 

Any module that follows that protocol must implement a `to_string` function.  Both strings and integers do.  More complicated types like tuples do not.

A list will work:

```elixir
iex> sample_list =['a','b','c','d']
['a', 'b', 'c', 'd']

iex> IO.puts('Hello, #{sample_list}')
Hello, abcd
:ok
```

A tuple will not:

```elixir
iex> t = {1, 'a'}
{1, 'a'}

iex> IO.puts('Hello, #{t}')
** (Protocol.UndefinedError) protocol String.Chars not implemented for {1, 'a'}
    (elixir) lib/string/chars.ex:3: String.Chars.impl_for!/1
    (elixir) lib/string/chars.ex:17: String.Chars.to_string/1
```

In this case, you can work around things by specifying `inspect` instead of relying on Elixir to use `to_string`:

```elixir
iex> IO.puts('Hello, #{inspect t}')
Hello, {1, 'a'}
:ok
```

## Inspect This

{% img http://variousandsundry.com/cs/images/elixir-inspect_colored_lettered.jpg %}

That looks a little magical, until you dig a little further and realize how it works:
  
```elixir
iex> IO.puts('Hello, #{Kernel.inspect(t)}')
Hello, {1, 'a'}
:ok
```

`inspect`, like `to_string`, is not a magic keyword.  It's [a function in the Kernel module](http://elixir-lang.org/docs/stable/elixir/Kernel.html#inspect/2).  The Kernel module is automatically loaded every time you run Elixir.

If you're not far enough down the rabbit hole yet, look closer at `Kernel.inspect/2`.  Yes, that's right: an arity of 2.  That second parameter is a list of options that become an `Inspect.Opts` struct, which is part of the Inspect protocol.  

And, yes, [we're going there next](http://elixir-lang.org/docs/stable/elixir/Inspect.Opts.html).

`Inspect.Opts` gives you some interesting choices. For example, you can `limit` the number of values returned. For example:

```
iex> IO.puts('Hello, #{Kernel.inspect(t, [limit: 1])}')
Hello, {1, ...}
:ok
```

More drastically:

```
iex> listicle = ['a', 'b', 'c', 'd', 'e', 'f']
['a', 'b', 'c', 'd', 'e', 'f']
iex> IO.puts('Hello, #{Kernel.inspect(listicle, [limit: 1])}')
Hello, ['a', ...]
:ok
```

Seems kind of weird to me that the absence of the rest of the values is shown with an ellipses, but I'm sure that comes in handy somewhere.

There are also options like `:pretty`, which enables pretty printing, and `:width` that limits the number of characters in a string line when pretty printing is invoked.  For example: 

```
iex> IO.puts('Hello, #{Kernel.inspect(listicle, [limit: 3, width: 3])}')
Hello, ['a', 'b', 'c', ...]
:ok
iex> IO.puts('Hello, #{Kernel.inspect(listicle, [pretty: true, limit: 3, width: 3])}')
Hello, ['a',
 'b',
 'c',
 ...]
:ok
```

You can, of course, see all the parameters [in the Inspect.Opts documentation](http://elixir-lang.org/docs/stable/elixir/Inspect.Opts.html).

OK, let's work our way back up to the polymorphism I promised earlier.

## Polymorphic Protocol


`String.chars` is [a protocol](http://elixir-lang.org/docs/v1.0/elixir/Kernel.html#defprotocol/2).  That is, it creates a set of formulas that anyone following the protocol must create for their particular module if they want to proclaim themselves as conforming to a specific protocol.

In `String.chars`' case, there's only one function to make happen: `to_string()`. Whatever structure you're using, to be sure there's a way for it to be represented on screen or in a log is for it to hew to this protocol, which is just a convention that has to be uniquely implemented for a given library.

So, multiple modules will have a `to_string` function, and it'll work the same everywhere because they conform to the protocol.

With `String.chars`, you can print the values of certain variable types out to a string, such as in the example we started this whole essay with.  `IO.puts` can only print the value of a variable out to the screen as long as the variable is of a type that can be converted to a string, either through `String.chars` protocol or `Kernel.inspect`.  The `#{}` is just shorthand for all of that.

 
## To Sum It Up

IO.puts prints stuff to the screen, and you can include variables in that.

When you interpolate a value into a string, you're using a couple of Kernel modules, adhering to a protocol, and being all polymorphic.

Thankfully, you didn't go with  the <> operator, instead, because then we could travel down a fun road of macro programming that would take another thousand words to get through. 


## Completely Unrelated

In my research for this Core Elixir installment, I came across a website with ["Hello World" examples in dozens of languages](http://c2.com/cgi/wiki?HelloWorldInManyProgrammingLanguages).

The 'winner' of the page is a tie between the Whitespace language and [the Obfuscated Perl example](http://www.perlmonks.net/index.pl?node_id=329174). Nobody comes close to either of those.

Really, that Perl example will blow your mind.

## Scheduling Notice

I'm off to [OSCon](http://www.oscon.com/open-source-2015) next week, so Core Elixir will be taking a week off.  It'll be back the following week.  Thanks for not weeping openly at this news.

_If you have any comments, questions, complaints, criticisms, or corrections, catch me on Twitter, [@AugieDB](https://twitter.com/augiedb).  That handle is the same as my GMail account, if you need to type more characters. I want these articles to be factually correct and will update them as necessary._

_(7)_
