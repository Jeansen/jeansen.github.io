---
layout: post
title: bcrm
feature-img: "assets/img/portfolio/bcrm.svg"
img: "assets/img/portfolio/bcrm.svg"
date: 2018-11-1T20:08:11+01:00
tags: [github, projects, bcrm, linux]
---

With the end of the year 2017 I had decided to setup a raspberry pi running some central services for my home infrastructure.
While doing so I felt the need for some automated backup. So, I searched around, but did not find what I was looking for. 
In the end I hacked together my own script.

Throughout my search I cam across dump and restore. This duet looked very promising, but is no longer maintained. 
So I resorted to tar and rsync.

Having a script that copies one system to another turned out to be not that hard. But it became more complex when I 
wanted to have support for LVM.


With the end of the year 2018 not that far away the "script' grew beyond my original expectations and is able to do a lot more than just creating a clone of a SD card for a raspberry pi.

Therefore I decided to rename it from pi2clone to bcrm as in (Backup, Clone, Restore and More)

Check out the [project page on github](https://github.com/Jeansen/bcrm)!