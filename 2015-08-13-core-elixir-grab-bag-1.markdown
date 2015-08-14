---
layout: post
title: "Core Elixir: Grab Bag 1"
date: 2015-08-13 22:23:07 -0400
comments: true
categories: elixir Elixir coreelixir CoreElixir
---

# Self-Evident Git Commit

They say your Git commits should be descriptive.

[This commit](https://github.com/elixir-lang/elixir/commit/cf41811462d4d7e819aea73f3a33adb082fab9d4) added backticks to surround true and false 64 times across 30 files. The git log on it reads, "add backticks to a massive amount of true, false and nil." 

"Massive" is a good word.


# Alias Assumption

When you use an Alias in a module, you don't have to specify what to Alias it as -- if you just want to use the last section of the module's name. 

For example, if you have a module named `TV.Series.Name`, the default alias will be 'Name'.  These two commands do the same thingL

```elixir
alias Long.Module.Name, as: Name
alias Long.Module.Name
```

# Renaming Maps

Elixir works hard to keep the arguments of its function calls consistent, for the sake of the pipeline operator. But it doesn't stop there.

The Map library has a couple of functions that re-order the arguments for that sake, but also rename the function for readability.  For example:

```elixir
def has_key?(map, key), do: :maps.is_key(key, map)
```

It's not enough to simply re-order the arguments there.  The function name, itself, is changed to make it more readable and consistent with the rest of Elixir's syntax.  Read it out loud and I think you'll agree that, in that order, the "has_key" name works better.  "'Map' has key 'key'" reads better.  

Since you'll likely be including the module name with a call to this function, there's even a certain rhythm there:

```elixir
Map.has_key?(map, key)
```

Alternating map/key/map/key.

And, because Elixir allows for it (as in Ruby), the question mark in the function name indicates that the return value will be a boolean.  That's not enforced by the language. It's in the source code's specs, but that's only useful for documentation and tools like Dialyzer.  It's a stylistic thing that you're encouraged to use.

If you do declare a function that ends in a question mark that returns an integer, prepare yourself for a pull request to fix that...

   
# Erlang is Not For the Feint of Heart

The Supervisor shutdown variable is a bloodthirsty one.  You can choose between [three values when shutting down a Supervisor](https://github.com/elixir-lang/elixir/blob/4f68c4f10502e0f54a21093bb9a33957e63a9ac4/lib/elixir/lib/supervisor/spec.ex#L139).  

First, there is `:infinity`.  This is the most forgiving option, leaving the child process that is another Supervisor all the time it needs to stop everything below it first before the Supervisor dies off.

Second, there is a number, which represents the time it'll wait for the child process to shutdown.  If it doesn't get confirmation back from the child that it shut itself down in  _x_ number of milliseconds, the Supervisor takes out the child hard:

  Process.exit(child, :kill)

There aren't too many languages out there that  give you the chance to kill your children like this.

But, wait, there's more!  There's a _third_ option:

  :brutal_kill

Erlang is starting to sound like either an 80s action movie or a 90s fighting video game.  That option goes straight to killing the child without spending any time waiting. Just -- boom!  The child is out of its misery without warning.

Some might call that more humane.
