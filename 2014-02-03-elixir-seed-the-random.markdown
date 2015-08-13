---
layout: post
title: "Elixir: Seed the Random"
date: 2014-02-03 00:45:28 -0500
comments: true
categories: erlang elixir random seed
---
Just a quick note for something I was just reminded of the hard way:

Always seed your random values.

[If you're using the ```Enum::Shuffle/1``` function](http://elixir-lang.org/docs/master/), for example, pay attention to the part of the documentation that says:

>Notice that you need to explicitly call ```:random.seed/1``` and set a seed value for the random algorithm. Otherwise, the default seed will be set which will always return the same result.

Do as the documentation suggestions and use this:

>:random.seed(:os.timestamp)

Otherwise, you *will* get the same results returned to you a thousand times.

Trust me. I know. I've lived it.  Save yourself.

_(Special thanks to [@sfalcon](https://twitter.com/sfalcon/status/430183996740206593) for reminding me...)_

__Update August 11, 2015__: `Erlang.now` [is deprecated in Erlang as of v18.0](http://www.erlang.org/news/85).  The text of this post has been updated to change to the `:os.timestamp` command to set the random seed.
