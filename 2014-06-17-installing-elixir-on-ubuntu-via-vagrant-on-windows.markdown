---
layout: post
title: "Installing Elixir on Ubuntu via Vagrant on Windows"
date: 2014-06-17 21:39:57 -0400
comments: true
categories: elixir ubuntu windows vagrant
---

##Installing Elixir, Hiding Windows

I'm heading off to [OSCon](http://www.oscon.com/oscon2014) and [Elixir Conference](http://elixirconf.com) next month.  These will be my first tech conferences ever, and I'm very excited.

One small program is that I don't have a laptop of my own anymore.  My old Mac laptop (pre-unibody construction) failed last year.  My only choice is to bring my work machine, which is a Thinkpad Windows box.

So as not to be the laughing stock of either conference, particularly OSCon, I decided to install Vagrant and run that at full screen size at all times and pretend I have a Linux box.  Also, this gives me Elixir access at work, where I plan to someday introduce the language.

For posterity's sake, here's how I did it:

__Warning: This is something of a down and dirty guide. For the sake of a temporary installation, it works.  If I were setting up a new server that I'd keep up permanently in the long term for production purposes, I'd likely make very different decisions.I wouldn't take some of the shortcuts I take here.__

First, [install Vagrant](http://www.vagrantup.com).  That's outside the scope of this write up, but it's very easy.  Follow the link for details.

I installed the basic Ubuntu machine that the on-line docs use as their "Get Started" example.  Once I set up [Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/) to be my client to ssh into my new Vagrant machine, I got to work on installing Erlang and Elixir. 

##Start with Erlang, Of Course

First, you'll need some basic tools to set up Erlang. Install a bunch:  

	sudo apt-get install build-essential libncurses5-dev openssl libssl-dev fop xsltproc

This will give you little things you'll need, like `make`.  I still can't believe the default Ubuntu box doesn't come with that, but who am I to judge?

Now, bring down the latest Erlang source code.  As I write this, it's at 17.0:

	wget http://erlang.org/download/otp_src_17.0.tar.gz

Then unzip it:

	tar zxvf otp_src_R17.0.tar.gz

Change directories and install away:

	cd otp_src_R17.0
	./configure && make && sudo make install

If the box still complains that MAKE isn't installed, try this:

	sudo apt-get install build-essential 

That ought to take care of it.  Then try again.

Go to the Erlang REPL to make sure it's installed:

	$ erl
	Erlang/OTP 17 [erts-6.0] [source] [async-threads:10] [kernel-poll:false]

	Eshell V6.0  (abort with ^G)
	1>

Success!

##Time for Elixir

I went back to my home directory and downloaded the latest and greatest version of Elixir. In this case, that's Elixir 0.14.1-dev:

	cd
	wget https://github.com/elixir-lang/elixir/archive/master.zip

This is when I discovered the hard way that Ubuntu also doesn't come with `zip` installed, so:

	sudo apt-get install unzip

Elixir relies on UTF8.  Go ahead and type 'locale' to see what locale your box is set to.  The Vagrant Ubuntu box is not set up as UTF8, which will lead to a bunch of installation errors.  So let's cover our tracks here first.  I went a little overboard, probably, but I did this to set it up for my `vagrant` user, who is the only user who will ever use this system:

	vi /home/vagrant/.pam_environment

And put this in that file:

	LANG="en_US.utf8"
	LANGUAGE="en_US.utf8"
	LC_CTYPE="en_US.utf8"
	LC_NUMERIC="en_US.utf8"
	LC_TIME="en_US.utf8"
	LC_COLLATE="en_US.utf8"
	LC_MONETARY="en_US.utf8"
	LC_MESSAGES="en_US.utf8"
	LC_PAPER="en_US.utf8"
	LC_NAME="en_US.utf8"
	LC_ADDRESS="en_US.utf8"
	LC_TELEPHONE="en_US.utf8"
	LC_MEASUREMENT="en_US.utf8"
	LC_IDENTIFICATION="en_US.utf8"
	LC_ALL=en_US.utf8

You'll have to log out and back in again for the new location to set in.  After that, change directories back to your Elixir directory and run away:

	cd elixir-master
	make clean
	make test

Then you need to make sure the `iex` REPL is in your PATH:
	
	PATH="$PATH:/home/vagrant/elixir-master/bin"

And you're all set:

	$ iex
	Erlang/OTP 17 [erts-6.0] [source] [async-threads:10] [kernel-poll:false]

	Interactive Elixir (0.14.1-dev) - press Ctrl+C to exit (type h() ENTER for help)
	iex(1)>

Away you go!

Some resources I used in setting this all up:

* [Installing Vagrant](http://docs.vagrantup.com/v2/installation/index.html)
* [How To Install Make on Ubuntu](http://askubuntu.com/questions/161104/ubuntu-how-do-i-install-make)
* [Erlang Downloads Page](http://www.erlang.org/download.html)
* [Basho's Erlang Installation Guide](http://docs.basho.com/riak/1.3.0/tutorials/installation/Installing-Erlang/#Installing-on-GNU-Linux) was very handy.  That's basically where I cribbed my Erlang installation directions from.
* [Elixir's Getting Started Guide](http://elixir-lang.org/getting_started/1.html)


