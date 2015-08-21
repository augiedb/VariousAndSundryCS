---
layout: post
title: "Core Elixir: Grab Bag 1"
date: 2015-08-13 22:23:07 -0400
comments: true
categories: elixir Elixir coreelixir CoreElixir
---
{% img http://variousandsundry.com/cs/images/elixir_grabbag.jpg %}

In the course of researching "Core Elixir" articles, I come across all sorts of little points of interest that don't always fit into the article, no matter how far I stretch them. Those wind up in a little scrap file.  That scrap file becomes the Grab Bag.

## Self-Evident Git Commit

They say your Git commits should be descriptive.

[This commit](https://github.com/elixir-lang/elixir/commit/cf41811462d4d7e819aea73f3a33adb082fab9d4) added backticks to surround true and false 64 times across 30 files. The git log on it reads, "add backticks to a massive amount of true, false and nil." 

"Massive" is a good word.


## Alias Assumption

When you use an Alias in a module, you don't have to specify what to Alias it as -- if you just want to use the last section of the module's name. 

For example, if you have a module named `Long.Module.Name`, the default alias will be 'Name'.  These two commands do the same thing:

```elixir
alias Long.Module.Name, as: Name
alias Long.Module.Name
```

I'm still wrapping my brain around writing modules with multiple levels like that, so this is a neat trick for me.

## Renaming Maps

Elixir works hard to keep the arguments of its function calls consistent, for the sake of the pipeline operator. But it doesn't stop there.

The Map library has a couple of functions that wrap an Erlang function directly, re-ordering the arguments for the pipeline operator's sake, but also to rename the function for readability.  For example:

```elixir
def has_key?(map, key), do: :maps.is_key(key, map)
```

It's not enough to simply re-order the arguments there.  The function name, itself, is changed to make it more readable and consistent with the rest of Elixir's syntax.  Read it out loud and I think you'll agree that, in that order, the "has_key" name works better.  " 'Map' has key 'key' " reads better than however-you'd-read-the-Erlang equivalent.  ("'Maps' 'is a key' named 'key' in 'map'"?!?)  It's obvious what it does, but it doesn't read as cleanly.

Since you'll likely be including the module name with a call to this function, there's even a certain rhythm there:

```elixir
Map.has_key?(map, key)
```

It alternates between Map and Key.

And, because Elixir allows for it (as in Ruby), the question mark in the function name indicates that the return value will be a boolean.  That's not enforced by the language. It's in the source code's specs, but that's only useful for documentation and tools like Dialyzer.  It's a stylistic thing that you're encouraged to use.

If you do declare a function that ends in a question mark that returns an integer, prepare yourself for a pull request to fix that...

   
## Erlang is Not For the Faint of Heart

The Supervisor shutdown variable is a bloodthirsty one.  You can choose between [three values when shutting down a Supervisor](http://elixir-lang.org/docs/v1.0/elixir/Supervisor.Spec.html).  

First, there is `:infinity`.  This is the most forgiving option, leaving the child process that is another Supervisor all the time it needs to stop everything below it first before the Supervisor dies off.

Second, there is a number, which represents the time it'll wait for the child process to shutdown.  If it doesn't get confirmation back from the child that it shut itself down in  _x_ number of milliseconds, the Supervisor takes out the child hard:

```elixir
  Process.exit(child, :kill)
```

In the words of Elixir's documentation, "the child process is unconditionally terminated". Gruesome!

There aren't too many languages out there that  give you the chance to kill your children like this.

But, wait, there's more!  There's a _third_ option:

```elixir
  Process.exit(child, :brutal_kill)
```

Elixir is starting to sound like either an 80s action movie or a 90s fighting video game.  That option goes straight to killing the child without spending any time waiting. Just -- boom!  The child is out of its misery without warning.

Some might call that more humane.

## Housekeeping: Updating Core Elixir

[I made an update to the Collection To List installment](http://variousandsundry.com/cs/blog/2015/07/30/core-elixir-collection-to-list/) that added a new section, cleaned up some confusing code, and just generally rambled on.  

I also [cleaned a couple of minor things up on [last week's System.tmp_dir post](http://variousandsundry.com/cs/blog/2015/08/06/core-elixir-system-dot-tmp-dir-slash-0/).

Further back, I made an update to a January 2014 post about random number generation to reflect a recent deprecation in Erlang.

But the big news is, [this blog is now on Github](https://github.com/augiedb/VariousAndSundryCS)!  I've taken the original Markdown files for every post and created a Github project for them.  If you see any problems with past, present, or future posts here, make a pull request.  (Spelling errors, coding errors, what have you.)  If there's a part of Elixir you'd like to see me tackle in this series, raise an issue.
	
_If you have any comments, questions, complaints, criticisms, or corrections, catch me on Twitter, [@AugieDB](https://twitter.com/augiedb). Or [make a pull request on Github](https://github.com/augiedb/VariousAndSundryCS)!  That Twitter handle and Github ID is the same as my GMail account, if you want to deal with it more quietly. I want these articles to be factually correct and will update them as necessary._

_(10)_
