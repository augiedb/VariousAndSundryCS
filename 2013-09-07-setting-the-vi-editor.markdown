---
layout: post
title: "Setting the vi Editor"
date: 2013-09-07 17:03
comments: true
categories: 
---
*(Here's a fine example of why people blog: To write something down so they don't forget it and always have a handy reference back to it.)*

Special thanks to [Michal Taszycki](https://github.com/mehowte), who pointed this out to me over on Exercism.io:

If you're using the vi or vim editors (and I'm on vi currently), there are options you can set.  Save them to a file named ```~/.exrc``` and vi will run with them everytime you start it up.

It's huge to set up the vi editor to a way that makes you productive and comfortable.  

One of the things I've battled most is the old tabs and spaces problem. Should a TAB be a single character or x number of spaces?  If the latter, how many?

I'm firmly in the camp that the TAB should be translated into spaces.  How many spaces is up to you, though the Ruby community seems to think it should only be 2.  I'm going to go with that, though it looks a little too flat to me.  I think 3 is better, but I don't want to pick too many nits here. Also, since the Ruby community so favors flatter code structures, I wouldn't think there'd be a great need to make for smaller indents.  It's not like well-factored Ruby code is going eight levels deep and running off the screen, right?

In any case, this is the link I'm getting to.  It's the ["Tabs and Spaces" VimCasts.org screencast](http://vimcasts.org/episodes/tabs-and-spaces/). It explains all the relevant options, including one or two I didn't know even existed.  I'm going to stick with ```set ts=2 sts=2 sw=2 expandtab```

That last part is important, as it tells the editor to fill in the two spaces the TAB button skips over with two actual space characters.  That seems to make all the difference in the world when you share the code elsewhere.

