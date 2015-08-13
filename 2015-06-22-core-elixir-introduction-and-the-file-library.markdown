---
layout: post
title: "Core Elixir: Introduction and the File Library"
date: 2015-06-22 21:48:30 -0400
comments: true
categories: Elixir coreelixir file core
---


Welcome to __Core Elixir.__

I've been busy studying [Elixir's source code](https://github.com/elixir-lang/elixir) lately. 

I've never looked too deeply into any language's source code.  Perl and Ruby scared me off with their C origins, I guess. 

Looking more closely at the Elixir source code, though, I'm Iearning a lot.  The code is easy to understand, clearly documented, and not too scary.  Granted, I haven't gone into the parts where stuff is compiled or macros are created or -- well, the _deeper_ stuff.  I'll save that for a rainy day. (_Monsoon_ rainy, I should think.)

The outer libraries, though, are fair game.

That's what "Core Elixir" will be -- a series of articles looking at the core libraries of the Elixir language:  How they work, what fundamentals of the programming language they expose, where Erlang fits into all of this, how it compares to other languages like Ruby and Perl (because I know something about those two), and more.

And, occasionally, you can watch my frustration or ignorance boil over into something approaching madness and disbelief at the way things work.

The biggest thing I've noticed so far is easy to sum up:

# Elixir is Elixir

__I'm amazed at how much of Elixir is written in Elixir.__  You can see smaller building blocks being written in Elixir, and then the slightly more complicated ones building themselves on top of those smaller bits.  And when Elixir doesn't have the answer, Erlang _does_ and can be called out to. It's a remarkably efficient way to create a language that improves on what's there and doesn't just copy it as an exercise.  If the latter were true, Elixir wouldn't be nearly as popular as it is today.

I've paid the most attention recently to the Elixir `File` module. As a Perl programmer, being able to manipulate files in various ways is of special interest to me.  So much of the work I've done in my career has been in automating things that this module does the individual parts of. I can't help but look here first.

# The File Libraries

__Elixir's `File` library is mostly a wrapper around Erlang's `File` module.__  Or, as we Elixir folks like to call it, `:file`.  The effect is so strong that `:file`, itself, is aliased at the very top of the Elixir File module to save a lot of typing, like so:

```elixir
  alias :file, as: F
```

Seriously, it's the second line of non-commented code in the module.  It's just that important.

# What's In a Name?

Elixir creator, Jose Valim, has spoken out in that past on the importance of Elixir being about more than just wrapping Erlang functions.  So what's he adding here, specifically?  From what I've seen, the biggest addition to the language that this module gives us is the ability to think in more Unix-y terms.

Erlang's file manipulation code is much more florid in its prose.  It's descriptive, but not intuitive to a modern programmer, necessarily. Erlang has function names like `get_cwd` and `set_cwd` instead of `cwd` and `cd`.  Elixir's `stat` is Erlang's `read_file_info`.  It may feel like Erlang's syntax here is more expressive and plainer in its English, but __Elixir utilizes the programmer's built-in knowledge of the Unix commands__.  As a programmer, you don't need to translate Unix to Elixir, or learn a second set of commands to do the same things as the command you already know.  The Unix and Elixir commands are identical, for the most part.  

# Code Pattern

Elixir has private functions, which are defined with a `defp`.  Those, as that description might suggest to you, aren't available to the programmer using the library.  

The `File` library uses it in an interesting way. The regular function that's publicly accessible has the simple and memorable name.  The work that it does, however, is done in a private function that's preceded by `do_` in its name.

For example, if you want to to create a specific directory, you call on the `mkdir_p` function with the path name.  Elixir will create that directory as well as any of the parent directories along the way.  Plain old `mkdir` will handle creating only a single directory without the option for the parent directories.

Here's what that function looks like:

```elixir
def mkdir_p(path) do
  do_mkdir_p(IO.chardata_to_string(path))
end
```

The only thing it does is call out to a private function of similar name, translating the path name along the way to something Erlang would find more palatable.  (We'll get to the whole string versus list of characters things in a future installment, I'm sure.)

You've asked it to run a command, and now Elixir will `do_` that command.

From there, Elixir can pattern match to handle the request across multiple versions of the code.  In this case, there are two:

``` elixir
  defp do_mkdir_p("/") do
  defp do_mkdir_p(path) do
```

_Spoiler:_ The first one returns `:ok` and does nothing, as that directory already exists.  The second one actually creates a new directory with the Erlang function `:file.make_dir`.  

Funny enough, in writing this article, I found that this command didn't use the `F` alias like every other call to `:file` did in the program.  I submitted a pull request on that over the weekend which got merged in. ( [Use file alias 'F' for consistency #3412](https://github.com/elixir-lang/elixir/pull/3412) )

You know how people tend to like short pull requests? That one deleted 5 characters and added 1.   This beats [my previous shortest pull request](https://github.com/elixir-lang/elixir/commit/4c2252d7ddd927b6c8821eacadf930e91237a8d) by a few characters. Only a few, though...

I <3 open source.

# Coming Soon...

`File.stat` -- Or, "inode  for Elixir".  Or, `File#stat` for Elixir.  Or, even `:file.stat` for Elixir.

After that, well, there's lots of stuff ready to go: Collections and lists,  Enumerables, dates and times, odds and evens, why `IO.puts` is the simplest and most complicated thing ever, where in Elixir There's More Than One Way To Do It, a diversion into documentation formatting, and so very much more...

_If you have any comments, questions, complaints, criticisms, or corrections, catch me on Twitter, [@AugieDB](https://twitter.com/augiedb).  That handle is the same as my GMail account, if you need to type more characters. I want these articles to be factually correct and will update them as necessary._

_(1)_
