---
layout: post
title: "Core Elixir: File.stat"
date: 2015-06-24 21:37:14 -0400
comments: true
categories: Elixir coreelixir core file stat
---

_(Note: Do not confuse the function `File.stat` with the module `File.Stat`.  Note the capitalization on the second word there.  We'll be covering that one in the near future.)_

`File.stat`, at its heart, is a struct that holds the filesystem information of a given file.

If you have any experience with programming on a Linux or Unix box, a lot of this struct will look very familiar to you.  Its functional equivalent is the command line `stat` command:

```
[AugieDB] ~ $ stat index.html
16777218 26895214 -rw-r--r-- 1 augiedb staff 0 4437 "May 15 23:53:44 2015" 
"Aug  9 22:47:11 2012" "Aug  9 22:47:11 2012" "Aug  9 22:47:11 2012" 4096 
16 0 index.html
```

Those values include the file's permissions, size, the file system id, and more. You can, of course, `man stat` for further information at the command line.


## The Source

In the Elixir world, that command is rewritten as `File.stat/2`.  Other languages have similar things.  [Perl](http://perldoc.perl.org/File/stat.html) and [Ruby](http://ruby-doc.org/core-2.2.2/File/Stat.html), for example, both have their own `File::Stat` modules.

Erlang has [File.read_file_info](http://www.erlang.org/doc/man/file.html#read_file_info-1) in [its File library](http://www.erlang.org/doc/man/file.html#read_file_info-1).  It populates a data type named [`file_info`](http://www.erlang.org/doc/man/file.html#type-file_info) with all the information.  That's what Elixir calls on for the sake of `File.stat`.  It then converts it from Erlang's data structure to a native Elixir struct via `File.Stat.from_record/1`. (Again, we'll talk more about that next time.)

In the end, your Elixir request gets a chunk of data like this in response:

```elixir
 %File.Stat{access: :read_write, atime: { {2015, 5, 15}, {23, 53, 44} },
    ctime: { {2012, 8, 9}, {22, 47, 11 }, gid: 20, inode: 26895214, links: 1,
    major_device: 16777218, minor_device: 0, mode: 33188,
    mtime: { {2012, 8, 9}, {22, 47, 11} }, size: 4437, type: :regular, uid: 501} }
```

Of the three languages I've mentioned in this post, I like this one the best.  It just gives you all the results along with the descriptive names in one big shot.  You can parse/pattern match and transform all you like from there.  No need to remember which values come in which order, or what the naming convention for the new object is.  It's all laid out for you right there in the returned value.


## The Options

You can pass along an option list to go along with that file path.  You don't get a lot of options, but they might come in handy for your time-keeping needs:

```
stat_options :: [{:time, :local | :universal | :posix}]
```

Basically, you can format the time stamp of the file in whichever way you'd like.  Let's run all the options against the same file and see what the differences are.

We'll start with the default case, which is `local`.  If you want local, you don't need to even specify it.  (Thus, 'default'.)  Leave the options blank and just pass in the filename.  In the interests of your curiosity, though, here's what it looks like when you include the option:

```elixir
iex> f = "welcome.html"
iex> File.stat(f, [time: :local])
{:ok,
 %File.Stat{access: :read_write, atime: { {2015, 5, 15}, {23, 53, 45} },
  ctime: { {2014, 2, 16}, {21, 51, 46} }, gid: 20, inode: 50093003, links: 1,
  major_device: 16777218, minor_device: 0, mode: 33188,
  mtime: { {2014, 2, 16}, {21, 51, 46} }, size: 2839, type: :regular, uid: 501} }
```

The date/time stamp is structured as `{ {YYYY, MM, DD}, {HH, MM, SS} }`.

I'm on the east coast of the United States, so that's a 9:51:46 p.m. time stamp on the `ctime` and `mtime` there.
 
'Universal` gives back the Greenwich Prime Meridian time:
 
```
iex> File.stat(f, [time: :universal])
{:ok,
 %File.Stat{access: :read_write, atime: { {2015, 5, 16}, {3, 53, 45} },
  ctime: { {2014, 2, 17}, {2, 51, 46} }, gid: 20, inode: 50093003, links: 1,
  major_device: 16777218, minor_device: 0, mode: 33188,
  mtime: { {2014, 2, 17}, {2, 51, 46} }, size: 2839, type: :regular, uid: 501} }
```

Now you see it's the following day at 2:51:46 a.m. 

And `posix` gives you back that ridiculously long series of numbers that only a computer could love -- or someone doing math with seconds:

```
iex> File.stat(f, [time: :posix])
{:ok,
 %File.Stat{access: :read_write, atime: 1431748425, ctime: 1392605506, gid: 20,
  inode: 50093003, links: 1, major_device: 16777218, minor_device: 0,
  mode: 33188, mtime: 1392605506, size: 2839, type: :regular, uid: 501}}
```

That time stamp equates to 'run it through Google and hope for the best.'  ([Here's one.](http://www.epochconverter.com))


## Bang!  

There's a variation on `File.stat`, too, with `File.stat!`  The difference is that exclamation point at the end.  It's an Elixir convention that a module ending in an exclamation point will not return a tuple, but rather the information you requested, or an error.

For example:

```
iex> File.stat('File does not exist.md')
{:error, :enoent}

iex> File.stat!('File does not exist.md')
** (File.Error) could not read file stats File does not exist.md: no such file or directory
    (elixir) lib/file.ex:282: File.stat!/2
```

In the source code, `File.stat!` calls `File.stat` to get the job done, then pattern matches on the results to return back the appropriate `File.error` type if necessary, or just pass back the bare info.

The error codes are taken from POSIX error code standards. Here is a small sampling of those:

* __enoent__ - [Error No Entry](http://stackoverflow.com/questions/19902828/why-does-enoent-mean-no-such-file-or-directory) (i.e. no file)
* __eacces__ - You don't have the right access rights (permissions) to that file
* __eisdir__ - Not a file. Might be a directory. Or a device. 
* __enotdir__ - Not a directory
* __enospc__ - No space.  The drive/disk partition is full.
* __eperm__ - Operation Not Permitted (You can't do that; it doesn't make sense.)
* __efbig__ - Your file is too effing big. 

(For more details on the difference between "eperm" and "eacces", [check out this article](http://www.comicbookdb.com/graphics/comic_graphics/1/679/32616_20150118104424_large.jpg).  Thanks for the link, [Jan](https://github.com/jandudulski)!)

You can find more errors listed in [the Erlang File module documentation](http://www.erlang.org/doc/man/file.html).  See if you can guess what they all mean, then compare them to [this ridiculously exhaustive list](http://www.ioplex.com/~miallen/errcmp.html) that will make your eyes bleed.  

And may your bathroom never have an __epipe__ error...

__Coming next:__ We'll talk about the `File.Stat` module.  It's different from the `File.stat` function, I swear.

_If you have any comments, questions, complaints, criticisms, or corrections, catch me on Twitter, [@AugieDB](https://twitter.com/augiedb).  That handle is the same as my GMail account, if you need to type more characters. I want these articles to be factually correct and will update them as necessary._

_(2)_
