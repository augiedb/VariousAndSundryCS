---
layout: post
title: "My Podcast App: The Main Interface"
date: 2013-10-14 23:30
comments: true
categories: iOS, podcasts
---

This is what I came up with for my podcast app's main play screen:

{% img http://variousandsundry.com/cs/images/podcast_1.png 'iOS Podcast Player Play Screen' %}

The first thing you'll likely notice is the lack of buttons.  That's for a very good reason. This app is designed to be used on the road, and to help me skip over commercials and tedious show openings.  I don't want to have to look at the screen to do that. It's gesture-based.  Swipe left to right to fast-forward by 30 seconds.  Swipe right to left to rewind 15 seconds.  Tap to pause. Tap again to play.

iOS 7 causes a bit of a problem with this. If you start your fast-forwarding swipe too far from the left, it merely brings you back one screen in the application. Since I use UINavigationBar for navigation, that functionality comes standard with it. I believe there's a flag to turn that behavior off, but I haven't had the time to look for it and implement it.

Overall, it's a fairly simple looking screen.  It's not perfect, but it suits my purposes.  

Looking back on the process now, I'm amazed at how many little decisions go into making this page:

* The bar at the bottom of the screen is not a scrubber. It's just a thermometer bar type thing to tell you how far along the podcast you are visually.  You can't grab a vertical bar in the middle and rewind or fast forward great distances. Only 30 or 15 seconds at a time with a swipe.  It's a limitation of the program that I've never hit up against, so I'm cool with it.

* In iOS 6 and earlier, the bar was much thicker.  Made it easier to read. With iOS 7, it became an itty bitty line, and less noticeable.  Annoying, that.

* The bar is black when playing and turns red when you pause the playback to give you some visual indication of the state the podcast is in. That visual indicator was much bigger in iOS 6.  In iOS 7, maybe I need to turn the whole background of the screen red?  If the point is to make for an easy to use app while otherwise occupied, large signals like that would be key.

* Above that, I have two numbers: Time elapsed and total time of podcast.  This is one I went back and forth over.  First, where to position it?  Left, right, or center? Above or below the bar? Should the second number be the total time, or the time left till the end?  Should the size of the two numbers be different to give one data point additional importance?  Should the numbers turn red when the podcast is paused, too?  (Yes, they should, but it's an idea I just had while writing this up.)  The code updates the timer a couple times per second, as I recall. Is that fast enough?  Too fast? Too slow?  I don't know.  I'm not too nit-picky about whether I'm 45:31 into a podcast, or 45:32.  So no problem there.

* Is the podcast album art too large?  Too small?  Should it be the background and everything else be over top of it?  Should I fade it out then?  

* The album art doesn't always show up.  Sometimes, it even shows up in the menu screen (that I haven't shown yet) but not on the play screen.  That's most annoying. Haven't looked into why that is yet, though.  We'll look at that menu screen another time.

* The show title above the album art takes some regex trickery to remove the redundant podcast name from the title of the episode. It's not perfect.  That's freeform text, so people can adjust those titles in any way they want, whether it's compatible with my code or not.

* I like having the podcast date up there.

* Problem: No show notes.  I like having those, but there's no room on the main screen.  Maybe it could have a button to pop those up, but I don't want to start adding buttons to the screen, which is meant to be one large tap target.
 
* I don't like the back arrow in the upper left in either iOS 6 or iOS 7. It needs something custom that I haven't had the time to figure out.

Right now, this project isn't under active development.  It's "good enough" for what I need it to do.  I learned a lot from doing it, and maybe that's enough for my purposes. We'll see.

