---
layout: post
title: "Installing Elixir 0.13"
date: 2014-02-21 23:58:00 -0500
comments: true
categories: elixir, map, erlang, beta
---

If you want to play with maps in Elixir, you need to install the bleeding edge of both Elixir and Erlang.  I did it this weekend.  Here's how I did it.

## First, There Shall Be An Erlang

First, you need to install Erlang/OTP 17-RC1.  It took some digging, but [here's the link.](http://www.erlang.org/download/otp_src_17.0-rc1.tar.gz)

(Release Candidate 2 is due out this week, so that link might be outdated by the time you read this. I'll update this post when RC-2 comes out.)

Go to that directory and unzip it:

		$ tar -xvf otp_src_17.0-rc1.tar.gz

Change into the directory that was just created for you:

		$ cd otp_src_17

-- and install Erlang:

		$ ./configure  [options are available, but I didn't use any]
		$ make
		$ make install

You may need to be installing this as root.  It will install files to parts of your system your user ID might not have access to.

I'm running OS X Mavericks.  I was able to get away with no parameters.  There are options you can use during configuration if you need to install things in specific places or if your libraries are elsewhere. Read the documentation in the `HOWTO` directory to find out.  

To test that Erlang is installed, run its REPL, `erl`:

		$ erl
		Erlang/OTP 17 [RELEASE CANDIDATE 1] [erts-6.0] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]

		Eshell V6.0  (abort with ^G)
		1>

Bingo!


## And Then There Was Elixir

Now, install Elixir from the v0.13 branch off Github. I'm not an advanced Git user, so [I consulted StackOverflow and found the answer](http://stackoverflow.com/questions/1911109/git-clone-a-specific-branch) to cloning a specific branch of a repository.  Pretty simple:

		$ mkdir elixir13
		$ cd elixir13
		$ git clone -b v0.13 https://github.com/elixir-lang/elixir.git
		Cloning into 'elixir'...
		remote: Reusing existing pack: 56247, done.
		remote: Counting objects: 339, done.
		remote: Compressing objects: 100% (337/337), done.
		remote: Total 56586 (delta 172), reused 0 (delta 0)
		Receiving objects: 100% (56586/56586), 21.30 MiB | 1.76 MiB/s, done.
		Resolving deltas: 100% (28061/28061), done.
		Checking connectivity... done

(You can do this from any directory you have access to, really.  It's up to you.  I'd name it something sane, though.)

That puts all the files into an "elixir" subdirectory, so go one more level deep and let 'er rip:

		$ cd elixir
		$ make clean test

Mine tested cleanly. The README suggests running `make clean` and `make test` if yours has issues.

Spin up the Elixir REPL while you're there to see where everything is at:

		$ bin/iex
		
		Erlang/OTP 17 [RELEASE CANDIDATE 1] [erts-6.0] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]

		Interactive Elixir (0.12.5-dev) - press Ctrl+C to exit (type h() ENTER for help)
		iex(1)>

## Text Expander for Me, Maybe BashRC Alias for You?

At this point, though, just running `iex` would run the old 0.12.3-dev version I had installed first.  I think as a temporary fix, I'll invoke [Text Expander](http://smilesoftware.com/TextExpander/index.html) and not start remapping things in any dot files. I mapped `;iex` and `;elixir` to the files of the same semi-colon-less name in the `bin` directory of Elixir v.0.13.  This way it's only one extra character to get there and I don't have to think about it again.  If I need to find it, I can look it up in Text Expander.

Sure enough, it works and I'm updated.  I'll change directories into a current Elixir project I'm working on and run Mix.  (What you see here is how the line looks after Text Expander fills out my `;iex` shortcut.  Yes, my directory structure is a little haywire now.  I can deal with that for a month or so until things are more automated with Homebrew.)

		$ /Users/augiedb/erlang/elixir13/elixir/bin/iex -S mix

		Erlang/OTP 17 [RELEASE CANDIDATE 1] [erts-6.0] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]

		Compiled lib/shuffle_step/supervisor.ex

		lib/shuffle_step.ex:175: using // for default arguments is deprecated, please use \\ instead
		lib/shuffle_step.ex:175: using // for default arguments is deprecated, please use \\ instead
		lib/shuffle_step.ex:201: using // for default arguments is deprecated, please use \\ instead
		lib/shuffle_step.ex:201: using // for default arguments is deprecated, please use \\ instead
		Compiled lib/shuffle_step.ex
		Generated shuffle_step.app
		Interactive Elixir (0.12.5-dev) - press Ctrl+C to exit (type h() ENTER for help)
		iex(1)>

Deprecations ahoy!  It's a good sign that it's running, though. Let me run the mix test on Exercism's "Bob" program:

		[AugieDB] ~/exercism2/elixir/bob $ /Users/augiedb/erlang/elixir125/elixir/bin/elixir  bob.exs
		/Users/augiedb/exercism2/elixir/bob/bob.exs:31: warning: using % for sigils is deprecated, please use ~ instead

That was what I expected.  So that's good news!

Now I can start figuring out maps:

		iex(1)> user = %{ :name => "Augie" }
		%{name: "Augie"}
		iex(2)> user.name
		"Augie"

I'll start by changing my [Deck definition](http://elixirdose.com/shuffling-the-deck/) from records to maps.  Stay tuned for that. . .