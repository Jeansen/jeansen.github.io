---
layout: post
title: Disable KGpg autostart
date: 2022-07-10T01:33:52+02:00
tags: [linux, kde, gnome]
---

Today I received some updates for different KDE packages. I have KDE installed alongside GNOME, but do not use it very often. Anyway, Some longer time ago, I hat problems with KGpg popping up every time I logged in. I disabled it, but now it is there, again!

As it turns out, this seems to be an older [bug still open](https://bugs.kde.org/show_bug.cgi?id=373891) to be resolved. The fix is to simply add `OnlyShowIn=KDE;` to `/etc/xdg/autostart/org.kde.kgpg.desktop`.
