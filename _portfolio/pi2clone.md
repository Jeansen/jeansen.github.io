---
layout: post
title: pi2clone
feature-img: "assets/img/portfolio/pi2clone.svg"
img: "assets/img/portfolio/pi2clone.svg"
date: 2017-12-21T19:00:25+01:00
tags: [github, projects, pi2clone, raspberry pi]
---

With the end of the year 2017 I had decided to setup a raspberry pi running some central services for my home infrastructure.
While doing so I felt the need for some automated backup. So, I searched around, but did not find what I was looking for. 
In the end I hacked together my own script.

Throughout my search I cam across dump and restore. This tool duet looked very promising, but is no longer maintained. 
So I resorted to tar and rsync.

Having a script that copies one system to another turned out to be not that hard. But it became more complex when I 
wanted to have support for lvm.

Anyway, after some trial and error hacking, I finally got a script that is able to:

- backup a complete system to files
- restore a complete system from files
- clone one system to another (live)

The last point did work in the current implementation. But I will add support for lvm snapshots, too.

Check out the [project page on github](https://github.com/Jeansen/pi2clone)!