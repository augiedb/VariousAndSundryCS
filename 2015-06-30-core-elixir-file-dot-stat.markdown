---
layout: post
title: "Core Elixir: File.Stat"
date: 2015-06-30 09:29:45 -0400
comments: false 
categories: Elixir elixir coreelixir core file stat 
---

{% img  http://variousandsundry.com/cs/images/elixir_files_web.jpg   %}

[Last time, we talked about the 'File.stat' function.](http://variousandsundry.com/cs/blog/2015/06/24/core-elixir-file-dot-stat/)

This time -- not to be too confusing -- we're going to talk about the `File.Stat` library.  Notice the capitalization on "Stat".  I know, I have to double check myself every time I write it, too.

The `File.Stat` library is not something you'll likely use directly all that much, if ever, but it's very much necessary for the `File.stat` command.  There are all of two functions in the module, and they're both meant to translate the stat information between Erlang and Elixir, who use different types to hold the info.

* `File.Stat.to_record` will translate the information from an Elixir struct to an Erlang record.

* `File.Stat.from_record` will take an Erlang record and give you back an Elixir struct, which you can then match against and have all sorts of fun with.

Let's start by getting the file information directly from the Erlang `:file` module:

```elixir
iex> {:ok, f} = :file.read_file_info("index2.html")
{:ok,
 {:file_info, 3348, :regular, :read_write, { {2015, 5, 15}, {23, 53, 45} },
  { {2011, 7, 7}, {0, 52, 13} }, { {2011, 7, 7}, {0, 52, 15} }, 33188, 1, 16777218,
  0, 2556026, 501, 20} }
```

You'll see all the information in there, but it doesn't include the names as atoms attached next to them. Who knows which one is which?  (Yeah, you can look it up, but laziness is a virtue of programming, remember. [Thanks, Larry Wall.](http://threevirtues.com))

Instead, convert the Erlang file info (in its _record_ form) into an Elixir File _struct_ with `File.Stat.from_record/1`

```
iex> f_elixir = File.Stat.from_record(f)
%File.Stat{access: :read_write, atime: { {2015, 5, 15}, {23, 53, 45} },
 ctime: { {2011, 7, 7}, {0, 52, 15}}, gid: 20, inode: 2556026, links: 1,
 major_device: 16777218, minor_device: 0, mode: 33188,
 mtime: { {2011, 7, 7}, {0, 52, 13} }, size: 3348, type: :regular, uid: 501}
```

Ah, that's better.  We're back to the Elixir goodness there. When you use the `File.stat` function, Elixir handles both those steps for you.  You end up with the struct. In fact, the only other thing `File.stat` does for you is convert your Elixir character data into an Erlang string type. 

Going the other way: If you want to use an Erlang command to deal with that data, you can transform it back to Erlang's format:
 
```
iex> f_erlang = File.Stat.to_record(f_elixir)
{:file_info, 3348, :regular, :read_write, { {2015, 5, 15}, {23, 53, 45} },
 { {2011, 7, 7}, {0, 52, 13} }, { {2011, 7, 7}, {0, 52, 15} }, 33188, 1, 16777218,
 0, 2556026, 501, 20}
```

[This source code on this](https://github.com/elixir-lang/elixir/blob/v1.0.4/lib/elixir/lib/file/stat.ex) is also a classic case where there are twice as many lines to comment the code as there are lines of code.  It's worth it if you need to remind yourself of the definitions of things like `inode` and `ctime`.  Those Unix terms aren't always straight forward.

And judging from [this discussion on the Elixir-lang mailing list](https://groups.google.com/forum/#!topic/elixir-lang-core/dvk1Ak-_1Wo), there's room for improvement in the documentation here with the data types and whatnot.  Take this as a challenge, if you wish...

Core Elixir will return later this week to delve into the world of odds and evens.  It's never as simple as you may think...

_If you have any comments, questions, complaints, criticisms, or corrections, catch me on Twitter, [@AugieDB](https://twitter.com/augiedb).  That handle is the same as my GMail account, if you need to type more characters. I want these articles to be factually correct and will update them as necessary._

_(3)_


