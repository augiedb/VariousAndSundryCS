---
layout: post
title: "Learning vim"
date: 2014-03-02 14:26:56 -0500
comments: true
categories: vi
---

I'm a vi editor user kind of guy.  It's just the way I started in college, and is the way I've always felt comfortable.  I'm happy the Emacs users are happy, but it's not something I'm interested in converting to right now, thanks.

I'm getting serious about how I use vim lately, though.  I actually booted up the `vimtutor` for the first time in my life and am going through it. I've learned some neat things.  Most of it I already knew, but there are some little nuggets that I plan on using in the future.  Some examples:

 * _rx_

Replace the current letter with an 'x' (or use your own character here)

 * _cw_

Delete the rest of this word and go to insert mode

 * _:set ruler_

Gives you that neat line number/character position stat at the bottom of the screen always, as well as what percentage of the way into the file you are.

And here's where things get really embarrassing:

 * _:!_ 

Runs an external command.  I've always just saved out and run the command, or kept a second window open and switched over to it to run that script. But I can do it from inside the vim session, which is huge when doing test driven development.  Heck, it's so huge that I plan to start setting up my tests in my daily Perl programming in a better way to take advantage of this.

Then I'll simplify it with a keyboard shortcut either through Text Expander or the .vimrc, which is something else I'm still fiddling with.

But you can also grab a quick directory listing or cat a file or something else.  It auto-completes the file names, too.  Lovely.

 * _v_

Visual mode.  Command mode and Insert mode I had the hang of. Visual mode was completely foreign to me.  Being able to highlight a bunch of lines and then save them to a separate file? That's awesome.  To do that:

 * _:w filename_

...after you've highlighted what you want to save out.

Next up, I'm going to peruse some other folks' ```.vimrc``` files on Github, watch that [Ben Orenstein vim talk](http://confreaks.com/videos/1586-railsberry2012-write-code-faster-expert-level-vim), and dive into Derek Wyatt's [ridiculously large number of vim editor tutorial videos](http://derekwyatt.org/vim/tutorials/).




