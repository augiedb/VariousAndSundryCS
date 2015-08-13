---
layout: post
title: "TDD Rewires Your Brain"
date: 2014-09-02 23:49:14 -0400
comments: true
categories: perl testing tdd
---

While I've written tests in the past, I've done very little truly Test-Driven Development.

Recently, I had a small script to write for work that would be neatly self-contained, so I gave TDD a shot.  Quickly, it changed the way I program such scripts.

I didn't just power through, going step by step and writing each piece, testing it with a few print debug statements, and pressing forward. I wrote the tests alongside the code. Sometimes, I'd write the test first.  Other times, I'd get my code in place and then put together the test.  I know the true TDD Apostles just screamed at the latter option, but at least everything is getting tested.  

Since my first code attempts rarely worked, I was basically going red-green, anyway. ;-)

I broke things down into even smaller pieces than ever. I have nothing against writing a two line subroutine if it makes the code easier to read, but I do get a bit annoyed when I see a script with too many subroutines.  It's usually a trade-off there. A strong naming convention or style will certainly help there.

With TDD, though, it feels like more of a must-do thing. I want to test each step, so each step becomes a function.  What once would be one big subroutine is now a master subroutine that calls three distinct subroutines to handle the chunks of code that go into that routine.  It's much in the same way that the overall program works, where the main() calls the subroutines in the right order.

The smallest things get their own subroutines, too.  That simple regular expression or massaging of the data needs a test, because it's the most likely thing to have a small error in it you won't notice until you're much further down the line. If it needs a test and is a logical unit of code, it should get its own subroutine.

Doing this makes testing possible without copying-and-pasting bits of code, which are always subject to change. I have directories filled with little test scripts I've written to test out little ideas of code. Those are just messes. With TDD, all those Perl .t files are neatly contained and easily harnessed into one big test.

Nifty!

I'm also less hesitant to make big changes, because I know my tests are there to back me up.  Once I had a working program running, I immediately went back to refactor all those obvious things I ignored the first time around.  I'd run my tests as often as possible.  Make each change small, make it count, keep pushing ahead.

Now, there are tiny scripts I might need to automate things with in the future that I won't bother to TDD. If it's a one- or two-step process, it might not make sense.  In a perfect world, sure, I would.  In the real world, I'm much more pragmatic.

It feels like this TDD process took more time, but I also realize that a big part of that is the learning curve.  I would have made all the same mistakes the non-TDD way, too, but it might have taken me longer to realize it or to find the bugs.  With TDD, I could zero in pretty quickly on what was wrong. And with the proper tests, I'd find those errors more quickly than ever. I think that just wrapping my head around the TDD process and the extra code it produces is what cost me the most time.  With more practice and exercise, it'll become part of the drill and second nature.  The time thing will work itself out.

The trick now is in writing tests for pre-existing code bases without tests.  Most of those bits of code are monoliths without many subroutines. Ah, the joys of working with legacy codebases...


