---
layout: post
title: "Enjoying Android development through pain"
date: 2019-10-14 22:03:00 +0800
tags: coding android java
description: "My first impressions after a few weeks of writing an Android app"
image: "/assets/post_images/2019-10-14-pain-and-pleasure.jpg"
---

A few months ago I was sitting at my friend's house, as usual talking about programming and drinking tea. Somehow we came to a conclusion that why don't we build a mobile app. The friend was going to do a design, while I supposed to code. My coding enthusiasm stopped after I tried React Native and then Kotlin, could not choose whether I want to build a hybrid app or native (but actually I just got lazy and scared I can't make it).

Nevertheless, this semester I signed up for Mobile App Development course in NTU, decided to give it one more try. I did not have a choice of iOS or Android, hybrid or native, the choice was made for me by the course's curriculum: Android Java. And this is how my evenings with Android have started...

I can do web frontend, I thought. I can do backend. I should definitely have no problem with Android. I started with small things: got some understanding about activity lifecycle, gestures, scene transitions, intents, etc. Wrote a couple of small games. After that, inspired by my own imaginary success, I decided to do something slightly bigger: an application that gets some data from API and displays it.

First commit I committed on 29 September. Since then I have been writing code almost every evening and every free time during weekend. Guess, how far have I progressed so far? Only a couple of screens both of which are half way done. These are the screens:

<img src="{{ site.url }}/assets/post_images/2019-10-14-screen1.png" width="320"/>
<img src="{{ site.url }}/assets/post_images/2019-10-14-screen2.png" width="320" style="clear: right;"/>

These 2 weeks I went through despair, self-depreciation, frustration with Android overall and especially with Java, feeling myself miserable (but in between there were short moments of enjoyment). These are a few things that made me struggle:

- **XML Layouts**. Basically, in Android layout can be created by writing XML code or by using drag-and-drop interface. Drag-and-drop drove me crazy: elements did not look the same way in Emulator as they looked in Android Studio, elements did not want to move or moved by themselves whatever places they wanted. Writing XML code felt slightly better, but it is super verbose: just an empty screen with 1 button on it could be 15 lines of xml code.

- Got confused between **Constraint, Relative and Linear layouts**. Got confused even more when I found out that RelativeLayout is a legacy which was replaced by ConstraintLayout. How I supposed to know when do I use which? Until now no clue, but I guess it should come with practice.

- **Adding a list** to the screen and populating it with data from the API was a nightmare. At first I used a ListView. With the help of Stack Overflow was able to make it work. Then discovered that ListView is a legacy which is replaced by RecyclerView. Spent 2 days to replace ListView with RecyclerView (until now do not fully understand how my code works (especially RecyclerView Adapter) but it works).

- Making an element at the top of RecyclerView **scroll together with RecyclerView** took me another 2 days (until I found such a wonderful thing as NestedScrollView).

- Java itself feels very verbose. **570 lines of Java code** was written for these 2 screens (and additionally 454 lines of xml code by the way):
  <img src="{{ site.url }}/assets/post_images/2019-10-14-code-lines.png" width="640" style="display:block"/>
  Well, I can't complain about Java, it is what it is. But still, I feel like I wrote tons of code, but in reality there is nothing. It's like when you come to an expensive restaurant, pay a lot but come out hungry - this is how I feel after writing Java code.

Having said all of that, deep inside my heart I enjoyed all this time. It is not the same kind of enjoyment I have when writing Python or React.js code, but something different. The code is compiling, the app is not crashing, scroll is working - what more can I wish for?
