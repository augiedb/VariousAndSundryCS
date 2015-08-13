---
layout: post
title: "Baseball Processes"
date: 2014-02-20 23:21:55 -0500
comments: true
categories: elixir, processes
---
Here's Augie's Quick And Easy Guide to Running Elixir Processes:

	* You need to create a process.
	* That process will instantly send back its address (PID).  Remember it.  You'll need it later.
	* But that process won't send back a value yet.
	* You must then run the process. Pass any info it might need to its PID to get it running.
	* When the process is done, it sends the return value back.
	* Your main routine will then grab the return value and move along its merry way.

Let's strain an analogy to the breaking point:

It's like playing catch in the backyard. You don't want to play by yourself.  So you tell a friend to stand about 20 paces away from you. Once they're there, they wave their arms to be sure you can see them. Having seen them, you throw them the ball. They throw it back as a solid line drive right at your mitt. End of process.

Unless, of course, your friend is recursive. And wants another ball...

Maybe you want to practice catching pop flies, though. Tell another friend to stand over there somewhere. Call him "La Lob". Have him let you know where he is. Throw him the ball. He'll throw it back in a graceful arc.  And, assuming you made him to be recursive, he'll keep standing there, waiting for more, ready to send any balls back from whence they came in a graceful arc.

Wait, you want to field a ground ball?  Call up another friend. Tell him you want him to return all throws with a hard throw right at the ground towards your feet.  Send him 20 feet away.  Then toss him the ball. He'll turn around and throw it back at your feet.

Good news: You can throw three balls at once.  All three catchers will catch the balls and throw them right back at you in three different ways.  You'll pick up that list of balls on the ground and you'll know which one is which because that catcher's name will be on their ball.

You could also set it up so the balls lines up in the order they were thrown, in case order is important.

That's the fun of creating processes.  In baseball terms.  And a very stretched analogy.

Problem is, you've created three different friends. Wouldn't it be easier if you only needed one friend who could throw a line drive, a pop up, and a ground ball?  And when you throw them the ball, you also tell them how you want it thrown back? Sure, you could do that, too.  Enumerate over your baseball friends.

Once the team gets too big and you can't keep track of them all, draft a manager and let him throw the balls for you. You only wanted to catch the ball anyway, right?  Give him ten balls and tell him to surprise you with ten different return throws. Hell, let him determine how many friends to create. And if one trips and breaks a leg, tell him to bring in a new friend.

You're your own supervisor at this point.

And at some point, this all makes the Office of the Commissioner of Major League Baseball into OTP.

Play ball!

