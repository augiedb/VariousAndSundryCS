---
layout: post
title: "Core Elixir: Path.relative_to/2"
date: 2015-12-14 21:43:54 -0500
comments: true
categories: elixir Elixir CoreElixir coreelixir path
---

Sit back, this one has tangents on top of tangents.

(Don't I say that every time?)

(If you're tangential to a tangent, are you asymptotic?)

According to the documentation, `Path.relative_to/2` does this:

 > Returns the given `path` relative to the given `from` path.
 > In other words, it tries to strip the `from` prefix from `path`.

Sounds like fun.  Even better, it's recursive.  Eventually.

This function does not interact with the filesystem. It does it all via string manipulation, without any regexes. 

Should the path not contain the `from`, it returns the full path back to the user, like so:

```elixir
      iex> Path.relative_to("/usr/local/foo", "/etc")
      "/usr/local/foo"
```

Otherwise, it returns the relative path:

```elixir
      iex> Path.relative_to("/usr/local/foo", "/usr/local")
      "foo"
      iex> Path.relative_to("/usr/local/foo", "/")
      "usr/local/foo"
``` 

(I stole all those examples from [the source code documentation](http://elixir-lang.org/docs/stable/elixir/Path.html#relative_to/2).)


## But How Does It Work?

We start here:

```elixir
  def relative_to(path, from) do
    path = IO.chardata_to_string(path)
    relative_to(split(path), split(from), path)
  end
```

First, note that this is NOT an Erlang wrapper.  That always helps when writing Core Elixir. I'm selfish like that.

The first thing `Path.relative_to/2` does is translate the path argument to be a string.  Why's that?  Looking at the top of the Path source code's documentation, we see this:

```
   The functions in this module may receive a char data as
  argument (i.e. a string or a list of characters / string)
  and will always return a string (encoded in UTF-8).
```

It leaves the input options open, but standardizes on one particular output type.  That's smart. That gives the user maximal flexibility on input with a reliable return.  It lets the language do the work behind the scenes.

So how does `IO.chardata_to_string` do its thing?  Smells like a tangent...


## IO.chardata_to_string/1

First, it tests if it's already a string, and returns it unchanged if it is.  Thanks, guard clause!

```
  def chardata_to_string(string) when is_binary(string) do
    string
  end
```

If, on the other hand, it's a list of characters, it goes to an Erlang command, `:unicode.characters_to_binary/1` to handle the heavy lifting.  It returns the resulting string if all worked according to plan.  If not, it passes back an appropriate error:

```
  def chardata_to_string(list) when is_list(list) do
    case :unicode.characters_to_binary(list) do
      result when is_binary(result) ->
        result

      {:error, encoded, rest} ->
        raise UnicodeConversionError, encoded: encoded, rest: rest, kind: :invalid

      {:incomplete, encoded, rest} ->
        raise UnicodeConversionError, encoded: encoded, rest: rest, kind: :incomplete
    end
  end
```

I think that's fairly straightforward.  You get to use a `case` statement instead of a three way if-then-else, so it looks nicer, too.

Yes, this part of the function is an Erlang wrapper, but it has a lot of good error-checking afterwards added to it.

You might wonder what happens when the argument sent in is neither a list nor a string.  Fair enough.  

You get an error:

```
iex> c = HashDict.new
#HashDict<[]>
iex> IO.chardata_to_string(c)
** (FunctionClauseError) no function clause matching in IO.chardata_to_string/1
    (elixir) lib/io.ex:329: IO.chardata_to_string(#HashDict<[]>)
```

This makes sense.  Let it crash and all.  Give the code the two ways it might work, and then let it fail in any other.  The programmer is smart enough to take the hint from there.



## Back to the Function

Where were we again? Ah, here:

```elixir
  def relative_to(path, from) do
    path = IO.chardata_to_string(path)
    relative_to(split(path), split(from), path)
  end
```

`path` is now a string version of whatever was passed in, whether it was a string already or just a list of characters.

The next line (#3 above) splits apart the two paths that were passed in into their own lists of directories.  

"What's that?" you say.  "How do we split a path?"

Glad you asked. Please join me on our next tangent.


## Path.split

Elsewhere in the Path module, the `split` function takes in a path and breaks it apart into a list of its elements.  It uses a forward or backward slash as the separator, becuase it knows if you're on Windows and adjusts itself accordingly.  (Pro tip: It's always a good idea to use the language's built-in functions for dealing with paths for just that reason. It will allow your program to run anywhere.)

Here's `Path.split/1`:

```
 # Work around a bug in Erlang on UNIX
  def split(""), do: []
```

Wait, read that comment again.  __UNIX__ is the problem child here and not Windows?  Hunh.  Well, now I've seen everything.

But what if the path isn't empty?  It's not so easy.  Wait, actually, it is:

```
  def split(path) do
    FN.split(IO.chardata_to_string(path))
  end
```

`FN` is an alias for Erlang's `:filename` module.  We're letting Erlang do the work. 

I bet some of you are screaming "DRY" right about now.  We already talked about using `IO.chardata_to_string` earlier in the `Path.relative_to` function before we got to the line where we called `Path.split`.  Why now just let `Path.split` take care of that?

While it is a duplication of effort, we can't just go rewriting Core Elixir for this one function's sake.  And, hey, this is functional programming: We can perform the same function on a given piece of data a million times and always get back the same answer.  Also, the function will see that the path has already been stringified and pass it right back immediately.

I somehow doubt running this twice will result in a drag on the program's speediness.

When all is said and done, we end up with a path that's been split into a list of its directories, like in this example:

```
iex> File.cwd! |> Path.split
["/", "home", "vagrant", "augiedb", "elixir"]
```

Note that the first part is "/" to indicate that this isn't a relative path to where we are at the moment.  That's not some kind of indicator of what the directory split character is or anything.  You're welcome to use `Path.type` to see if the path is relative or absolute, but that's a topic for another time...

Also, I used `cwd!` in this example because plain old `cwd` returns one of those `{:ok, blah_blah_blah}' tuples, which can't be pipelined into `Path.split`.

Are we ready yet to get to the recursive part of `Path.relative_to`?  Yes. Yes, we are:


## Bring On the Recursion!

This should look familiar by now:

```elixir
  def relative_to(path, from) do
    path = IO.chardata_to_string(path)
    relative_to(split(path), split(from), path)
  end
```

`Relative_to/2` calls on `Relative_to/3` to do all the work.  The three arguments are (from left to right) a list of elements in the path that we're looking at, a list of elements in the path that we're starting from, and the initial path itself in plain string text. We'll see how that's used shortly.

The bulk of the work, assuming a long directory path, comes in the first pattern match on this function, and it's a little piece of pattern matching genius:

```
  defp relative_to([h|t1], [h|t2], original) do
    relative_to(t1, t2, original)
  end
```

The function pops the head off the list from both paths.  If they are the same (thus, the two "H"s in the parameters), we toss out that first item and do this again with the rest of the lists.  So "/usr/a" and "/usr/" would both match "/" as the heads and the function would get called again against "[usr/a]" and "[usr]".  It would run against those values and pull out the "use" items.  Then it would call itself with "[a]" and "[]".

In that case, where the two heads of the lists are now different (and the second one is already empty), it doesn't match this pattern and it bounces down to this pattern match:

```
 defp relative_to([_|_] = l1, [], _original) do
    join(l1)
  end
```

If the second value is an empty list, then take whatever is left from the first value and return it. There's your answer.

[_|_] would match to [a|[]].  In the second line, that list, as a whole, would be passed over to `Path.join' to -- well, become the rest of the directory after that shared portion.

If the difference were greater and we were looking at something like "/usr/a/b/c/d", then the match would leave you with "a", "b", "c", "d" and you'd join that list back up with Path.join to give you (in Unix world) "a/b/c/d".

I bet you're wondering how `Path.join` works, aren't you?  Sure, but that's a lesson for another day. 


## Path.relative_to

So far, we've covered two out of the three pattern matches that the three-arity version of `Path.relative_to` can handle.  We've seen what it can do when the first item in the path matches (it goes all recursive), and when the `path` has more items left than the `from` (time to return results), but what about when there are no lists left? It's just one value for the from and for the path?  What happens when the two paths are the same, or the 'from' path is longer than the `path`?

If we've come that far, we can't pull one path out from the other. Think about it: how can we pull a one item value out from a one item value?  It won't work.  We default to returning the original path, as was given to us earlier in the processs.

```
  defp relative_to(_, _, original) do
    original
  end
```

Since one can't be calculated from the other, we pass back the original value. For example: `/usr/a` can't be derived from `/usr/b`.  This function would run when passed the values for 'a' and 'b'.   If both paths are the same length, there's no way one will branch off from the other.


## How I Tested This All

Trying to wrap my head around the pattern matching in `Path.relative_to/3` meant extracting the code for that function and testing it separately.  I created a file named `rel.exs` and copied the function's code into it.  I did some good old fashioned "IO.Puts Debugging" on this bad boy:

```
defmodule Augie do

 def relative_to(path, from) do
    path = IO.chardata_to_string(path)
    relative_to(Path.split(path), Path.split(from), path)
  end

  defp relative_to([h|t1], [h|t2], original) do
    IO.puts("first pm")
    relative_to(t1, t2, original)
  end

  defp relative_to([_|_] = l1, [], _original) do
    IO.puts("second pm")
    Path.join(l1)
  end

  defp relative_to(a, b, original) do
    IO.puts("third pm")
    IO.puts "-- path param: #{a}"
    IO.puts "-- from param: #{b}"
    original
  end

end
```

I name the module after myself, again, because I'm an egomaniac and I know it won't have namespace issues with core elixir.  (I doubt a pull request to create an "Augie" module would get past Jose...)  I could have named it "Patha" or something, but I'm pretty fast with typing my own name.

I tested in iex, calling the script as a parameter to `iex`:

```
vagrant@precise32:~/augiedb/elixir$ iex rel.exs
Erlang/OTP 18 [erts-7.0] [source] [async-threads:10] [kernel-poll:false]

Interactive Elixir (1.1.0-dev) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)>
```

And then I ran every possible scenario through it and saw where the debug statements came from:

```
iex(1)> Augie.relative_to('/opt', '/opt/a')
first pm
first pm
third pm
-- path param:
-- from param: a
"/opt"

iex(2)> Augie.relative_to('/opt/a', '/opt')
first pm
first pm
second pm
"a"

iex(3)> Augie.relative_to('/opt', '/opt')
first pm
first pm
third pm
-- path param:
-- from param:
"/opt"

iex(4)> Augie.relative_to('', '')
third pm
-- path param:
-- from param:
""

iex(5)> Augie.relative_to('', '/opt')
third pm
-- path param:
-- from param: /opt
""
iex(6)> Augie.relative_to('/opt', '')
second pm
"/opt"
iex(7)>
```

It's also somewhat magical when you type your name, hit tab, and a function is autocompleted after your name.

