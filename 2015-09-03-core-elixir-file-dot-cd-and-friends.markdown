---
layout: post
title: "Core Elixir: File.cd and Friends"
date: 2015-09-03 00:02:09 -0400
comments: true
categories: coreelixir elixir Elixir cwd cd bang baby 
---

We've talked briefly about the File library in the past when we discussed [File.stat](http://variousandsundry.com/cs/blog/2015/06/24/core-elixir-file-dot-stat/) and, of course and not confusingly, [File.Stat](http://www.variousandsundry.com/cs/blog/2015/06/30/core-elixir-file-dot-stat/).

This week, we return to the File library to look at how we might change the current working directory in Elixir.  It's pretty straightforward, but there's one neat twist to it.


## Change Your Directory

There might come a time in your code where you will want to change your working directory.   `File.cwd` will tell you what that directory currently is, but it is `File.cd/1` that will set you in a different directory.  You tell it the directory to change to, and it will either return an `:ok`, or an `:error` with the appropriate POSIX-compliant error message.

More succinctly, the official Elixir documentation says this:

```elixir
cd(Path.t) :: :ok | {:error, posix}
```

Same thing.

Here it is in action:

```elixir
iex> File.cd("/tmp")
:ok

iex> File.cwd
{:ok, "/tmp"}
```

Now, since this is Core Elixir, let's look at the implementation details.

## Yup, It's an Erlang Wrapper

The first clue that we're wrapping Erlang here is that the source code uses the alias `F`.  Looking at the top of the file, we see where that goes:

```elixir
 alias :file, as: F
```

Here's the code, then:

```elixir
  def cd(path) do
    F.set_cwd(IO.chardata_to_string(path))
  end
```

`:file.set_cwd` is [the Erlang command](http://www.erlang.org/doc/man/file.html#set_cwd-1) to _set_ the _current working directory_.  The only trick is that you have to stringify the path name so that Erlang will deal with it.  That's not so much of a trick as it is Elixir 101 at this point.

As usual, there is also the `cd!` version of the function.  Instead of just returning the error message and carrying on its merry way, it blows the whole thing up:

```elixir
iex> File.cd("/tmp/this_directory_does_not_exist")
{:error, :enoent}

iex> File.cd!("/tmp/this_directory_does_not_exist")
** (File.Error) could not set current working directory to /tmp/this_directory_does_not_exist: no such file or directory
    (elixir) lib/file.ex:1087: File.cd!/1
```

That latter error message appears in all red text, by the way, in case you questioned just how serious it was.

The source for this is, as you might expect, fairly similar to `File.cd`, but with an extra layer added on. (See lines 3 - 7, in particular.)

```elixir
  def cd!(path) do
    path = IO.chardata_to_string(path)
    case F.set_cwd(path) do
      :ok -> :ok
      {:error, reason} ->
          raise File.Error, reason: reason, action: "set current working directory to", path: path
    end
  end
```

The source stringifies the path name first, then runs `F.set_cwd` (which is also basically `File.cd` now) and pattern matches the results. If all is good, it returns `:ok`.  If something went horribly wrong, it pattern matches on the `:error`, prints out the reason , and blows stuff up.

It's just a case statement, though.  It's nothing we haven't seen before.


##One More Thing...

There's a `File.cd!/2` which I find the most fascinating function of all.  This one takes a directory and a function.  It changes into that directory, runs that function, and then resets your current working directory to where it was before you ran the command. 

That's pretty neat.

```elixir
iex> File.cwd
{:ok, "/home/vagrant/augiedb/elixir"}

iex> File.ls
{:ok,
 [".DS_Store", ".iex",  "card_game", "cards",  "cards.exs", "chapter13",  "dose",   "elixir", "Elixir.Card.beam", "Elixir.CardPoints.beam", "Elixir.Chip.beam",
  "Elixir.Factorial.beam", "Elixir.Gcd.beam", "Elixir.Guard.beam",
"factorial1.exs", "filechange", ...]}

iex> File.cd!("fileutils", fn() -> File.ls end)
{:ok,
 [".git", ".gitignore", "Augie", "config", "lib", "mix.exs", "README.md",
  "t.txt", "test", "_build"]}

iex> File.cwd
{:ok, "/home/vagrant/augiedb/elixir"}
```

You can see that we end in the same directory as we started, despite using a `cd` command in the middle there.  You can also see that there are two completely different directory listings given.

Let's see how it works:

```elixir
  def cd!(path, function) do
    old = cwd!
    cd!(path)
    try do
      function.()
    after
      cd!(old)
    end
  end
```

I almost laughed when I saw this code. It just strikes me as funny how terse it is.  It's the most succinct code I've seen in Elixir to date. Nothing fancy.  It's like a baby learning to speak. Everything looks like a two word sentence.

 > "Mama.  Dada.  Try do."

{% img http://variousandsundry.com/cs/images/babydrop.jpg %}

That's mostly because we're at a slightly higher level of abstraction here. This is `File.cd!/2` using `File.cd/1` to do the work of changing directories.  Much like with `File.cd!/1` using `File.cd/1`, you don't need to repeat all that code again for `File.cd!/2`.  It's already done for you; use it.  Plus, the function that `File.cd!/2` is running is neatly saved in a variable, `function`, so you don't have to look at all that code.  It's all building up on itself.

Here's how it works:

`File.cd!/2` first grabs the current working directory and saves it so it knows where to go back to later.

Then, it changes to the new directory, failing out if that doesn't work.  

Next, it runs the function before changing back to the original directory, no matter what. The `try do/after` construction just means that even if the first statement fails, the statement in the `after` section will still run.

Note that there is _no_ `File.cd/2` (without the `!`).  This is code that will either work, or blow up. There is no half measure on this one.

##One Last Piece of Trivia

`File.cd` and all of its variant arities aren't used anywhere else in the File library.

Hey, they call it "trivia" for a reason...

 _If you have any comments, questions, complaints, criticisms, or corrections, catch me on Twitter, [@AugieDB](https://twitter.com/augiedb). Or [make a pull request on Github](https://github.com/augiedb/VariousAndSundryCS)!  That Twitter handle and Github ID is the same as my GMail account, if you want to deal with it more quietly. I want these articles to be factually correct and will update them as necessary._

_(13)_