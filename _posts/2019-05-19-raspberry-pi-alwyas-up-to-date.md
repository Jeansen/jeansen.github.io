---
layout: post
title: Raspberry Pi - alwyas up to date
date: 2019-05-19T13:14:25+02:00
tags: [raspberry pi, linux]
---

By now, I have more than one Raspberry Pi installed at home. Most of the time I do not want to care about them because they simply serve as a one-time setup, similar to my router. After I have configured everything to my needs, I just expect it to work. I do not even want to worry about updates or upgrades. I simply want everything up to date all the time.

My solution for this on the Raspbarry Pi platform running Raspian is a simple combination of the two packages `needrestart` and `unattended-upgrades`. The former one will take care of all services that need to be restarted after any updates. The latter one will take care of automatic updates and upgrades (and restart the system, if necessary).

Because I have all my systems [setup with LVM]({{ site.baseurl }}{% post_url 2018-01-20-how-to-migrate-a-raspberry-pi-installation-to-lvm %}), there was one catch: Whenever a new kernel was installed I also had to [be prepared for new kernels]({{ site.baseurl }}{% post_url 2018-01-20-how-to-migrate-a-raspberry-pi-installation-to-lvm#be-prepared-for-new-kernels %}).
