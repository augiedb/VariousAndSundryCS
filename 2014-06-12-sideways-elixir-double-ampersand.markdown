---
layout: post
title: "Sideways Elixir: Double Ampersand"
date: 2014-06-12 22:32:29 -0400
comments: true
categories: elixir
---
*Sometimes, you can learn new things about a language when you squint your eyes just so hard, tilt your head at a jaunty angle, and see things sideways.  Today's tutorial is right up that alley.*

*Now straighten your neck.  You'll give yourself a pinched nerve like that...*

##Your Friend, Double Amp

This week, let's look at a basic part of Elixir that involves everyone's favorite and most beautiful typographic symbol, the ampersand.  Yes, even more people love the pipe operator, but that's two characters and doesn't count.  Nyah!  (Shut up, you freaky [interrobang](http://en.wikipedia.org/wiki/Interrobang) people...)

Everyone loves the ampersand. It might be hidden all the way down over the "7" key on your keyboard, but we love it anyway.  Font folks [go nuts](http://designshack.net/articles/typography/why-i-love-ampersands-you-should-too/) over them. Hell, Hoefler & Frere-Jones referred to the ampersand as their middle name, and you see [how well THAT turned out](http://nymag.com/news/features/jonathan-hoefler-tobias-frere-jones-2014-6/). Hoefler renamed the company [to preserve this blog post](http://www.typography.com/blog/our-middle-name), no doubt.  "Hoefler & Co."?  Yes, a "Co." dedicated to getting royally screwed over by his lies and lawyers.

{% img  http://www.variousandsundry.com/cs/images/hoefler_ampersand.png %}

But, really, the greatest Ampersand is the monkey from "Y the Last Man."  [Go read that book.](http://www.comicbookresources.com/?page=article&id=40318) 

{% img http://www.variousandsundry.com/cs/images/ampersand_monkey.png %}

But Wait!  One ampersand isn't good enough for this blog.  No, let's talk about two of them.  One might look at ```&&``` and think "short-circuiting operator."  If so, one has probably programmed before, and good for that one.

When used with two statements, Double Amp (as I like to call him) will return the first value if that value is false. The second one only returns when the first is true.  This is as confusing and as mind-warping as that first time you didn't pay attention and starting coding "unless not" conditionals and your brain burst.

##Any *Good* Tutorial Will Have Examples. And So Will This One.

Take this ridiculously contrived example. It's so ridiculous and rudimentary and ugly that it's the kind of thing only an insane tutorial writer could come up with. Nobody in their right mind would ever write something like this.

No, they'd probably make it into a macro.

		iex(14)> equals_five? = fn(x) -> x == 5 end
		#Function<6.106461118/1 in :erl_eval.expr/5>

Look, Mom!  I just wrote a lambda!  Or a closure! Or an anonymous function! I bet it's a monoid.  Nobody ever can explain those, so who's to tell me I'm wrong?

These Functional Folks are crazy.

{% img  http://www.variousandsundry.com/cs/images/romans_crazy.png %}

Let's test it out:

		iex(15)> equals_five?.(5)
		true

		iex(16)> equals_five?.(6)
		false

See?  It works!  It does what it says, so it doesn't need documentation! TEST-DRIVEN DEVELOPMENT FOR THE WIN!  (Except I wrote the tests after the code.  And DHH still hates it.)

While I'm full of self-loathing, let's see this function-so-ugly-not-even-Haskell-could-love-it in action:

		iex(17)> does_five_equal_five = equals_five?.(5) && IO.puts "This equals five!"
		This equals five!
		:ok

		iex(16)> does_five_equal_five
		:ok

Because the function is true, it checks the second part after the double-ampersand and executes it. In this case, it's not a boolean expression. It's just a command to print something out.  That command evaluates to being ```:ok``` after it's completed, though, so there's that.

		iex(18)> does_five_equal_five = equals_five?.(4) && IO.puts "This equals five!"
		false

		iex(18)> does_five_equal_five
		false

When the function is false, it never crosses the double ampersands to run anything.

This is an optimization. Why run the second part if you don't have to?  Both halves have to be true for the overall thing to be true, so once you've failed on the first part, why bother looking at the second?  It's a trick as old as time itself.  [Even Perl does it.](http://www.perlmonks.org/?node_id=301355)

The Double Amp allows you to basically guard the thing you really want to happen. Rather than ```if true then do```, you have ```true && do``` or ```false && never_do```.  

So the next time you're thinking that you want your code to look clean and follow logically and take great advantage of the wonderful pipe operator, think about Double Amp.  Then stop drinking and use the pipe operator.  It's a completely different thing.

Really? You thought about using Double Amp like that?  I'm taking your Microsoft Certification away from you!

__Next time:__ The Single Ampersand: For when writing ```fn(x) ->``` is just too much damned work....