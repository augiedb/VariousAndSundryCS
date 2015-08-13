---
layout: post
title: "Core Elixir: So You Say You Want to Copy a File"
date: 2015-07-10 00:19:08 -0400
comments: true
categories: coreelixir elixir core file copy
---

{% img http://variousandsundry.com/cs/images/elixir-copy_color_letters.jpg %}

To quote Perl, ["There's More Than One Way To Do It."](https://en.wikipedia.org/wiki/There%27s_more_than_one_way_to_do_it)

In fact, I'm counting **six**.  They come in pairs, though, thanks to our old friend the exclamation point.  Let's pair them off:

## File.copy and File.copy!

`File.copy` has the same exact name and argument order as the Erlang function that does the same thing.  Here, look at the source code:

```elixir
    def copy(source, destination, bytes_count \\ :infinity) do
      F.copy(IO.chardata_to_string(source), IO.chardata_to_string(destination), bytes_count)
    end
```
_("F" is an alias for the Erlang `:file` module.)_

It literally just converts the strings to the format Erlang prefers and runs the same function.

But what's with that `bytes_count` bit at the end?  I'll tell you, but let's first look at the return values of a successful call to `File.copy` and `File.copy!`:

```elixir
  iex> File.copy('_config.yml', 'testcopyfile_deletemenow.txt')
  {:ok, 3103}

  iex> File.copy!('_config.yml', 'testcopyfile_deletemenow.txt')
  3103
```

That number that comes back represents the number of bytes that were copied.  It is, effectively, the file size.

If you want to pattern match on the results of the copy, then you'll want to go with `cp` without the exclamation point at the end.

__`File.copy` has an arity of 3.__  Yes, there's a third argument.  If you don't include it, it defaults to `:infinity`.  Otherwise, it acts as a limiter and will only copy over that many bytes of the file to the new one.  This is part of the Erlang `:file.copy` function.

__Note:__ This version of file copying assumes you don't mind overwriting a file that already exists with the destination's name.  It will do so without warning.


## File.cp and File.cp!

[I said previously](http://variousandsundry.com/cs/blog/2015/06/22/core-elixir-introduction-and-the-file-library/) that one of the biggest helps the `File` module provides in wrapping the Erlang `:file` module is that it translates Erlang-speak to Unix speak.  If you want to copy a file, you can use the `cp` command with Elixir:

```elixir
    iex> File.cp('example.txt', 'copy_of_file.txt')
    :ok
```

And, assuming there are no errors, the `cp!` function will return the same thing:

```elixir
    iex> File.cp!('example.txt', 'copy_of_file.txt')
    :ok
```

As always with the bang, the error message is more human readable with it than without it, unless you're fluent in POSIX:

```elixir
    iex> File.cp('file_does_not_exist.txt', 'etc.txt')
    {:error, :enoent}

    iex> File.cp!('file_does_not_exist.txt', 'etc.txt')
    ** (File.CopyError) could not copy recursively from file_does_not_exist.txt to etc.txt: no such file or directory
      (elixir) lib/file.ex:439: File.cp!/3
```

## File.cp_r and File.cp_r!

Things get complicated here.

This is the recursive copy command.  If you provide it with a directory as the source, it'll copy everything in that directory to the directory destination you give it.

It returns the list of files it copied over, which can be extremely handy.

If you give it a single file name, it'll just copy that single file.

One important note for error checking's sake here: If the copy fails in the middle, it fails "dirty".  It'll leave what it had already copied into the source directory.  It's up to you to clean it up.

On success, it'll return a list of files and directories it has copied.  This includes the directory or directories you're copying into.

```elixir
iex> File.cp_r('tmp1', 'tmp2')
{:ok, ["tmp2/test3.md", "tmp2/test2.md", "tmp2/test1.md", "tmp2"]}

iex> File.cp_r!('tmp1', 'tmp2')
["tmp2/test3.md", "tmp2/test2.md", "tmp2/test1.md", "tmp2"]
```

I ran those commands back to back without clearing the second directory.  By default, it will just overwrite the files.  

You can, however, specify what to do if a file is already found in the destination directory. That's right, it's time for another case of __The Function with an Extra Arity__!  `File.cp_r` is `/3`.  The third parameter is a callback function to handle the source and destination file.  If that function returns true, the file will be copied.  If false, it won't be.

Here's how the function begins in the source code:

```elixir
 def cp_r(source, destination, callback \\ fn(_, _) -> true end) when is_function(callback) do
```

Things to notice here:
 * There's a guard clause on there to check that the third parameter is a function.
 * If no callback is given, there's a default function (to match the `is_function` guard) that gives a default `true` answer.

So if you want to default to a false answer, then pass a false function.  Make something fun up, like 

```elixir
  fn(_, _) -> false end
```

No, you can do better than that:

```elixir
  fn(_, _) -> 1 &&& 0 end
```

Impress your friends!  Confuse yourself six months from now with a misplaced sense of humor. Seriously, don't check that in. You'd be begging for a pull request.

I'll grab the example given in the documentation and change a couple of things to keep it consistent with what we've done so far:

```elixir
iex(1)> File.cp_r "tmp1", "tmp2", fn(source, destination) ->
...(1)> IO.gets("Overwriting #{destination} by #{source}. Type y to
...(1)> confirm.") == "y"
...(1)> end
Overwriting tmp2/test1.md by tmp1/test1.md. Type y to
confirm.y
Overwriting tmp2/test2.md by tmp1/test2.md. Type y to
confirm.y
Overwriting tmp2/test3.md by tmp1/test3.md. Type y to
confirm.y
{:ok, ["tmp2"]}
```

It appears that when you fall back to the callback, you lose the list of files you're copying.  If that matters to you, keep this in mind.

More fun: [Dig down deep enough in the code](https://github.com/elixir-lang/elixir/blob/v1.0.5/lib/elixir/lib/file.ex#L532) and you'll trip over our old friend, `Enum.reduce`.  Wait, `Enum.reduce` isn't your friend yet.  I'm writing these Core Elixir posts out of order.  We'll get there; you'll see.  Just remember that name in the meantime...



## Out of Context, This Looks Cray-Cray

```elixir
 defp do_cp_r(_, _, _, acc) do
    acc
  end
```

Is that Assembly Language? Elixir Morse Code?  Elixir Mad Libs? 

No, it's the default pattern match at the end of a list of matches, but its terseness out of context makes it look like utter gibberish.  Remember, kids, to __always read code in context__!  ("Code in Context" would make a great PragProg title... Someone else has "Code Complete," so...)


_If you have any comments, questions, complaints, criticisms, or corrections, catch me on Twitter, [@AugieDB](https://twitter.com/augiedb).  That handle is the same as my GMail account, if you need to type more characters. I want these articles to be factually correct and will update them as necessary._

_(6)_ 
