---
layout: post
title: "Core Elixir: System.tmp_dir/0"
date: 2015-08-06 23:33:33 -0400
comments: true
categories: elixir CoreElixir Elixir system temp directory coreelixir
---

_In which a Hex module is suggested, a pull request is considered, Erlang is explored, and a big warning kicks things off._


Sometimes, in the course of handling files through automation in any programming language, you'll have need of a temporary directory to stash things away in.  Elixir has something to help you find that directory: `System.tmp_dir/0` 

The command returns the name of the most suitable temp directory for you that is writable.

But the devil's in the details and it's very important to set expectations properly here.


## A BIG GIANT HUGE WORD OF WARNING
## Seriously.  Heed me.

{% img http://variousandsundry.com/cs/images/elixir_stop2_color_lettered.jpg %}

`System.tmp_dir` only _finds_ a directory for you.  It does not create one special to this program.  It will not guarantee for you that the directory is empty to begin with.  

It will merely give you a location on the file system where files can be written.  If you're lucky, there's an environment variable set to guide you to a good place.  If you're not lucky, it'll return `nil` and you won't have anywhere to go.  If you're supremely not lucky and are having a very bad terribly no good day, you'll get __the current working directory__ returned to you.

You see, my precious Perl has a module like this called [File::TempDir](http://search.cpan.org/~nanardon/File-Tempdir-0.02/lib/File/Tempdir.pm). (It's based on [File::Temp](http://search.cpan.org/~dagolden/File-Temp-0.2304/lib/File/Temp.pm), naturally.)  It creates a new and randomly-named temporary directory for you and then destroys it and everything in it once it goes out of scope.  It's a truly temporary place that's also self-cleaning.  Best of both worlds.

Elixir's `System.tmp_dir` will only help you find a directory that's writable. Period. It won't guarantee that other files won't be there.  It won't guarantee that what files you write to it will ever go away.  It won't destroy the temporary directory once it's gone.  That's still all on you.  So keep track of things.

You may consider this a warning, or you may consider this a golden opportunity to create a new application to put up on Hex for one and all.  Take your pick.

With that out of the way...


## Let's Look at the Code

`System.tmp_dir` takes no arguments and returns one string value: the location of the temporary directory you can write to.

```elixir
  def tmp_dir do
    write_env_tmp_dir('TMPDIR') ||
      write_env_tmp_dir('TEMP') ||
      write_env_tmp_dir('TMP')  ||
      write_tmp_dir('/tmp')     ||
      ((cwd = cwd()) && write_tmp_dir(cwd))
  end
```

This one is pretty simple.  The entire thing is a short-circuiting OR (||) statement.  The first chance Elixir gets to find something that will work, it'll exit out with that directory's location.

This function breaks up into three pieces:

 * First, it checks for environment variables. (lines 2-4)
 * Second, it'll check on a `/tmp` directory. (line 5)
 * Third, if all else fails, it'll go with the current directory. (line 6)

Let's drill down now and get our hands dirty at even deeper code:


## Looking Up the Directories

The three checks on the environment variables all use the function `write_env_tmp_dir/1`, taking as an argument the environment variable to check. 

Let's see how it works:

```elixir
defp write_env_tmp_dir(env) do
  case :os.getenv(env) do
    false -> nil
    tmp   -> write_tmp_dir(tmp)
  end
end
``` 

This uses the case statement, which is an Elixir macro short for "We need an IF statement but we're functional programmers and we don't use if statements, so cover it with a macro."

It farms the work out to Erlang, using its `:os.getenv` function to look for the system variable provided to it.

That's an interesting Erlang function by itself.  So let's take a look at it.  One more level down, we go:


## :os.getenv/0 - :os.getenv/2

[It comes in three flavors](http://www.erlang.org/doc/man/os.html#getenv-0), with arities of 0, 1, and 2.  The 0 arity function call returns all available environment variables.  Here's a truncated look at what that result is on my humble little Vagrant box:

```elixir
iex> :os.getenv
['LC_PAPER="en_US.utf8"', 'SSH_CONNECTION=192.168.33.1 51655 192.168.33.10 22',
 'PWD=/home/vagrant/augiedb/elixir', 'LC_ALL=en_US.utf8',
 'LC_IDENTIFICATION="en_US.utf8"', 'LC_MEASUREMENT="en_US.utf8"',
 'LESSCLOSE=/usr/bin/lesspipe %s %s', 'LC_NAME="en_US.utf8"', 'SHELL=/bin/bash',
 'ROOTDIR=/usr/local/lib/erlang', 'LC_MESSAGES="en_US.utf8"',
 'PATH=/usr/local/lib/erlang/erts-7.0/bin:/usr/local/lib/erlang/bin:/home/vagrant/.rakudobrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/opt/vagrant_ruby/bin',
 'LC_COLLATE="en_US.utf8"', 'TERM=xterm', '_=/usr/local/bin/iex',
 'LOGNAME=vagrant', 'BINDIR=/usr/local/lib/erlang/erts-7.0/bin',
 'MAIL=/var/mail/vagrant', 'LESSOPEN=| /usr/bin/lesspipe %s']
```

If you call it and pass along the environment variable (the /1 arity), it'll return the value:

```elixir
iex> :os.getenv('PWD')
'/home/vagrant/augiedb/elixir'
```

Finally, you can pass both a key AND a value into the function.  If Erlang finds that environment variable, it returns its value.  If that variable doesn't exist, it returns the value you passed in.  Do note, though, that the environment variable won't be created or modified.

```elixir
iex> :os.getenv('HELLO')  # Does not exist.
false
iex> :os.getenv('HELLO', 'WORLD') # Provide a default.
'WORLD'
iex> :os.getenv('HELLO')  # See?  It still does not exist.
false
```

## Where Were We?  Oh, Yes...

```elixir
defp write_env_tmp_dir(env) do
    case :os.getenv(env) do
      false -> nil
      tmp   -> write_tmp_dir(tmp)
    end
  end
``` 

We're past the case statement now.  You're getting 'false' in response to the `:os.getenv/1` command if the environment variable doesn't exist.  As a result, this function will return a `nil` back, which will force the next statement after the || (or) in the `tmp_dir` function above.

If there is a value for that key, you get that value back, which the code pattern matches with 'tmp' and sends as the argument to the next function down, `write_tmp_dir`. We have a directory, but now we'll check that we can use it!


## write_tmp_dir/1

This is the end of the road for our exploration.  It's as deep as we're going to get.  Honest.  This is where our final answer will be found and returned.

`write_tmp_dir` begins, amusingly, with [our old friend `File.stat`](http://variousandsundry.com/cs/blog/2015/06/24/core-elixir-file-dot-stat/)!  This time, it's looking for the properties of that temporary directory we're working on providing the coder, who's back about six steps now and doesn't realize it since things run so quickly on modern computer chips.

```elixir
defp write_tmp_dir(dir) do
  case File.stat(dir) do
    {:ok, stat} ->
      case {stat.type, stat.access} do
        {:directory, access} when access in [:read_write, :write] ->
          IO.chardata_to_string(dir)
        _ ->
          nil
      end
    {:error, _} -> nil
  end
end
```

Do `if` statements make you feel dirty?  How do nested case statements work for you?  Maybe I need to make a pull request to help flatten this out:

```elixir
defp write_tmp_dir(dir) do
  case File.stat(dir) do
    {:ok, stat} -> return_tmp_dir(stat, dir)
    {:error, _} -> nil
  end
end

defp return_tmp_dir(stat, dir) do
 case {stat.type, stat.access} do
  {:directory, access} when access in [:read_write, :write] ->
    IO.chardata_to_string(dir)
  _ ->
    nil
 end
end
```

That looks so much better to me.

`return_tmp_dir` is probably not the best name, but naming is hard and that isn't the point of this write-up.

I probably won't submit this as a pull request, though. Quoting [the Elixir Contributions docs](https://github.com/elixir-lang/elixir/blob/master/CONTRIBUTING.md):

<BLOCKQUOTE>
__NOTE:__ Do not send code style changes as pull requests like changing the indentation of some particular code snippet or how a function is called. Those will not be accepted as they pollute the repository history with non functional changes and are often based on personal preferences.
</BLOCKQUOTE>

I'm in a gray area here with this one.  This isn't a high holy war of camelCase versus snaked_names or parenthesis vs. no parenthesis, but it also doesn't add any functionality.  It looks prettier, I think, but I don't want to be a polluter. ;-)

As much as [I loved Garrett Smith's talk](https://www.youtube.com/watch?v=CQyt9Vlkbis), that doesn't mean I should apply his techniques everywhere I see Erlang/Elixir code. In __my__ projects, sure...

I've also looked at other Elixir source code and found a mixed bag of nested case statements.  A couple do, indeed, call out to another private function, but most just nest the code right there without that extra level of redirection.  (`File.do_cp_r` actually [goes three levels deep](https://github.com/elixir-lang/elixir/blob/master/lib/elixir/lib/file.ex#L565)!)

In any case, in plain English, the function grabs the stats on the directory. If it can't for some reason (the directory doesn't exist, for example), it won't match with `{:ok, stat}` and will instead return a nil.  Game over.  (Or, at least, on to the next check...)

If there are stats to be had, we go deeper. Let's pull those lines out specifically:

```elixir
  {:ok, stat} ->
    case {stat.type, stat.access} do
      {:directory, access} when access in [:read_write, :write] ->
        IO.chardata_to_string(dir)
      _ ->
        nil
    end
```

Here's a sample `File.stat` result

```elixir
iex(1)> File.stat('ia')                                                              
{:ok,                                                                                
  %File.Stat{access: :read_write, atime: { {2015, 7, 24}, {22, 35, 36} },
  ctime: { {2015, 7, 24}, {22, 35, 36} }, gid: 1000, inode: 112, links: 1,             
  major_device: 22, minor_device: 0, mode: 16895,                                     
  mtime: { {2015, 7, 24}, {22, 35, 36} }, size: 0, type: :directory, uid: 1000} }   
```

The code looks specifically for the `type` and `access` keys, pattern matching them in the next line to `:directory` and `access`.  In other words, you better be a directory and not, say, a file.  Secondly, there's a guard clause on line 3 to govern what type of access you need to have to that directory to use it.  It needs to be either read/write or write only.  That makes sense to have for a directory you want to write files into temporarily, don't you think?

If that all holds up, we get to our end position, which is that the function returns the writeable temporary directory name, properly stringified for peak Elixir usage.

Is That All?

No.

Don't be silly.

We haven't discussed what happens when the case statement back at the top leads to the `/tmp` directory.  You didn't forget about that while I was dragging your attention all around the world, did you?  

Can't say as I blame you...


## In Case of /tmp

It's been awhile, so let's go back to the very beginning:


```elixir
  def tmp_dir do
    write_env_tmp_dir('TMPDIR') ||
      write_env_tmp_dir('TEMP') ||
      write_env_tmp_dir('TMP')  ||
      write_tmp_dir('/tmp')     ||    ## You are here
      ((cwd = cwd()) && write_tmp_dir(cwd))
  end
```

We're up to line 5, where no environment variables have been set and so we look for the `/tmp` directory as a possible answer.  With this option, we can skip over the checks for environment variable status that we had with `write_env_tmp_dir` and go straight to checking on the `/tmp` directory at `write_tmp_dir/1`, a function we've already discussed.

It'll tell us if the directory exists, is a directory, and is writable.  Or not.

If not, we arrive at the final option.  


## Think Globally, Write Locally

{% img http://variousandsundry.com/cs/images/elixir_stop1_color_cwd_lettered.jpg %}

I call it the "nuclear option" for reasons outlined at the very top.  This is the default answer when all else fails: The current working directory.

Instead of calling out to another function, there's a little block of code with two commands joined by an AND (&&) to run the second half assuming the first half returns a value.  ([Erlang's documentation](http://www.erlang.org/doc/man/file.html#get_cwd-0) does bring up a circumstance under which this would fail -- if the directory's permissions prevent it.)  First, it sets the `cwd` variable up with the current working directory and, once that has a value, it goes back to the `write_tmp_dir` well to prove that it is a valid and writeable directory.

I'm not so sure why they didn't just combine it into one command like so:

```
 (write_tmp_dir( cwd() ) )
```

I tested it. It works.  Maybe it's less readable that way?  Is it a violation of Elixir style guides in that you're reading it inside out instead of left to right?  For one level of depth, I don't think it's a bad call. Plus, eliminated a temporary variables is always a pleasant thing.

What about the more Elixir-ish pipeline option:

```
  cwd() |> &write_tmp_dir/1
```

That looks like a jumble of punctuation, though.  I'm still not convinced the current line is the prettiest version of the code, but I'm really nit-picking at this point.  I definitely won't do a pull request here.


## One Last Note

There, is of course, a version of the function with the added bang to return a more verbose error in case of failure.  I present to you `System.tmp_dir!/0`:


```elixir
  def tmp_dir! do
    tmp_dir ||
      raise RuntimeError, message: "could not get a writable temporary directory, " <>
                                   "please set the TMPDIR environment variable"
  end
```

As you can see, it just calls on the bang-less `tmp_dir` function we've already exhaustively covered and, should it fail and receive a nil, it executes the second half of the OR (||) statement, raising an error complete with a suggestion for how to fix the problem. After all, the problem isn't the code; it's the file system. Set your ENV and be a better UNIX person.


## At Long Last, It's Summary Time!

There you have it -- the fancy pants Elixir way to find a temporary directory. Use with care.  Track the files you're playing with. Be a nice citizen and clean up after yourself.

And always eat your vegetables.

_If you have any comments, questions, complaints, criticisms, or corrections, catch me on Twitter, [@AugieDB](https://twitter.com/augiedb). Or make a pull request on Github! That Twitter handle and Github ID is the same as my GMail account, if you want to deal with it more quietly. I want these articles to be factually correct and will update them as necessary._

_(9)_

__Updated August 11, 2015__: Whops, got the arity slash in the wrong direction in one spot.  Also added clarification to the "AND (&&) to guarantee both halves will run" bit.  [Thanks to Henrik for the spots!](https://twitter.com/augiedb/status/629508059791032321)
