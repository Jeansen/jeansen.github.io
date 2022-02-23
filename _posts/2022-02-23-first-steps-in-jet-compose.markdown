---
layout: post
title: First steps in Jetpack Compose
date: 2022-02-23T20:26:28+01:00
tags: [Kotlin, compose]
---

In a previous post I wrote about my experience when converting Scala code to Kotlin. Now, I have some fine Kotlin code but this code still uses JavaFX. I liked JavaFX when it was all shiny and new. That is, when it in version 2.0. But, JavaFX never had - as far as I can tell - any big impact. AWT was a quick UI framework followed by Swing. The latter one is mature, but old. That does not make it bad. It simply misses a lot of new patterns in use today. JavaFX was "the new hope", but then Oracle cut it out! Now, there is a new player in town from Google: Jet Compose!

## Android for the Desktop

A long time ago ... 

OK, forget it. It's only been a couple of years when I started to play around with JavaFX in Java 8. I never liked AWT, nor Swing, nor SWT or JFace. Did a lot of stuff with all of these but never felt at home with them. Then there was JavaFX. So, I developed a little UI that I could utilize for my Router using the TR-064 protocol. Later, I converted what I had at that time to Scala. Code got reduced by almost half!

And, you named it, time is short and my list of interests is long. Other projects - which I gave much higher priorities - came along and years passed by. With the end of 2021 I decided to convert the Scala code base to Kotlin. Scala is a great language. I learned a lot with respect to functional programming and clean code during my Scala journey. Unfortunately,  I never had the chance to use it professionally.

So, with Kotlin I had a similar code base as I had with Scala. There were only minor syntactical changes and some library calls needed to be replaced. For instance in Scala XML is a first class citizen. With Kotlin I had to add a dependency to a third-party library. But that's about it!

And then there was JavaFX. Admittedly, it (still) works. But I think there is no real momentum anymore. Especially when it comes to designing and prototyping a UI. The tooling is simply missing.

Now, there is Jetpack Compose. A framework used for application development on Android. From some weeks, it is also available for the Desktop! Yes, it is only version 1.0.x and still has some bugs, but it works. And frankly, quite well. I first had to get used to the more functional DSL instead of the object-oriented DSL used in JavaFX. But I got my grip on this new framework ultra-fast! And the tooling is overwhelming. I really got fond of the live preview.

I am still at the beginning of my learning path. What I really like: There is no extra UI thread anymore. In JavaFX I always had to take care of the JavaFX UI thread and the Swing/AWT thread for my tray icon display. Another complexity I could get rid of. Theming is also a lot stronger than in JavaFX. And custom layouts are created with a hand-full lines of code.

I hope Jetbrains will keep this project alive longer than Oracle did with JavaFX. I think Compose has a very bright future. A lot of Android developers can now easily develop for desktop environments, too. And the web, of course!

So, I'll keep on learning. And when I have something to show, I'll publish it!


