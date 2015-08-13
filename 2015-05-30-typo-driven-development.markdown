---
layout: post
title: "Typo Driven Development"
date: 2015-05-30 15:00:41 -0400
comments: true
categories: TDD testing
---
I had one of those days:

<blockquote class="twitter-tweet" lang="en"><p>My code tests today are succeeding only in showing me all the typos in my tests. The code is fine. &#10;&#10;Isn’t that backwards somehow?</p>&mdash; Augie De Blieck Jr. (@augiedb) <a href="https://twitter.com/augiedb/status/581503839985098752">March 27, 2015</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Seriously, I spent more time debugging the tests than I did the code.  The good news here is that my code was pretty good on the first pass, which is rare. I just couldn't wrangle the tests together without every typo known to man sneaking in.

Frustrating.

<blockquote class="twitter-tweet" lang="en"><p><a href="https://twitter.com/augiedb">@augiedb</a> Have you written tests for your test-code yet?</p>&mdash; Meekostuff (@Meekostuff) <a href="https://twitter.com/Meekostuff/status/581543333786316800">March 27, 2015</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

The kinda actually serious answer to this is, yes.  I compile my code before I run it in Perl.  I have a vi shortcut, so all I have to do is type "cc" to compile the code and see if there are any syntax errors.  Then I type "rr" and it runs for me.

I'm somewhat proud of that workflow. It's the little things that make me happy, I guess.

One last tangent: "tt" is the shortcut I use to run PerlTidy against the code.  I don't accept all the suggestions, but it's good enough to make me happy.  I know I can modify the changes it looks to make, but I haven't gotten the energy up to do that research yet.  Laziness is a virtue, right, Larry Wall? ;-)
 
