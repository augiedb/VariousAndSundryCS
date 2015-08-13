---
layout: post
title: "The Problem With Perl Refactoring"
date: 2013-08-30 22:44
comments: true
categories: 
---
Refactoring is an easy concept, ultimately.  You're rewriting code to make it cleaner or faster or more clear.  The end result is still the same, but the way you get there is different.

(Yes, that's a vast simplification, but that's all we need relevant to this post.)

Refactoring code involves finding the weak parts of the code and rewriting them.  Having a suite of tests to back you up on this is a good thing.  Heck, it's a necessary thing.

Ultimately, though, knowing how to refactor code comes from two things: Language knowledge and experience.  You can refactor code all day and go in circles.  It's the knowledgeable refactorings that bring added value to the code base.

That's why I bristled so much in watching [this "Perl Refactoring" video](http://www.youtube.com/watch?v=gIsc39vU22w).  There's a serious problem in the Perl community and that's its over-reliance on the CPAN for everything. Searching for a module to refactor code for you is silly. Refactoring takes a knowledge of code.  In the case of Perl, that might mean using a module from CPAN in lieu of some hairy code you wrote yourself but that is potentially unstable and not time tested.  But looking for a CPAN module that will point out what to refactor, or even hoping such a beast would do the refactoring for you is just silly.

Here's the key to refactoring: You need to know the language and know the concepts behind it.  No automated refactoring machine will save you.  *You* have to do the work.
