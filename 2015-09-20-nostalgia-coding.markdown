---
layout: post
title: "Nostalgia Coding"
date: 2015-09-20 22:02:40 -0400
comments: true
categories: C64 coding 1980s commodore
---
I came across [this wonderful page](http://mocagh.org/loadpage.php?getcompany=usborne-hayeshttp://mocagh.org/loadpage.php?getcompany=usborne-hayes) tonight that has PDF versions of the classic books from the 80s written to teach kids how to code using sample games.  The code was mostly the same across the Commodore, Atari, and other computers of the time.

I had a few books like this growing up, and can remember having a couple of these specifically, at one point or another.  (Maybe a library loan?)

This is the first block of Commodore 64 BASIC I've looked at in a while, and I almost laugh when I see some of this stuff and how we used to do things.  (And some things were so close to how we do them today, with only minor differences.) For example:


{% img http://variousandsundry.com/cs/images/basic_sample.png %}

1. Line numbers.
2. GOSUB to line numbers instead of subroutine names. (I don't remember this at all, but typing "GOS" and holding the shift key down for that last letter was a shorter version of "GOSUB" that worked.  Wow.)
3. Variable names where the sigil (_$_) came _after_ the variable name, not before.
4. Tests of equality with a single equal sign instead of double. (This is a mistake that still catches programmers today.)
5. Separating commands on a single line with a colon instead of a semi-colon.
6. The IF statements! My goodness, the IF statements!  I'm looking at one program where 115 lines in a row are IF statements.  Functionally, I believe they're IF THEN ELSE, but there was no ELSE keyword, so you wound up checking them all, anyway.
7. Looks like all variables are globals, and most are named with single alphabetical characters.
8. ```PRINT CHR$(147)``` clears the screen.  That's muscle memory.  I may not have typed it in nearly 30 years, but I recognized it instantly when I saw it in the code.
9. Courier looks like a futuristic font by comparison to what these books show the code as. This code looks almost like it came straight off the dot matrix printers where you needed to tear the sprockets off the side of the pages...
10. "LET" was not necessary on the C=64, which is why it looks so unusual to me here.  Today, "LET" is the keyword you use in Swift to define a constant.

In twenty years, I'm sure I'll look back at today's Ruby or Perl or Swift and laugh at it, too.  C'est la vie.


