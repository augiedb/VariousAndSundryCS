---
layout: post
title: "Simple Elixir Pleasures"
date: 2014-08-15 20:48:47 -0400
comments: true
categories: elixir
---

You know that part of a blog where there's __x__ number of comments and your job, as the programmer, is to properly label them with "comment" or "comments"?  It's an annoying little bit of code, usually involving another if/then sequence.  You can sorta shortcut it if you want to look cheap by saying "comment(s)" and saving yourself one check.  

Bill C, [in his Redditex repo](https://github.com/billc/redditex), has [a better solution](https://github.com/billc/redditex/blob/master/lib/redditex/item.ex
), thanks to Elixir's pattern matching of functions:


```ruby
    defp comment(0), do: ""
    defp comment(1), do: "(1 comment)"
    defp comment(n), do: "(#{n} comments)"
```

I love that.  It's so clear and so clean looking to me.

Sigh. I'm turning into one of those one jerks who hates if/then/else.