---
layout: post
title: From Raspberry Pi 3 to 4
tags: [raspberry pi, linux]
date: 2019-08-31T17:25:31+02:00
---

When I received the new Raspberry Pi 4 (with 4GB RAM), I thought it would be as easy as taking the SD card from a Raspberry Pi 3 and put it in the new version 4. Well, close but not exact. You will have to copy over some additional files for the boot partition. I managed this by simply taking the actual available Raspbian image, extracting it to a new SD card, booting it up (for verification) and then copying over the complete boot partition.

This should be enough in the generally. Of course, in my case - with an unconventional [LVM setup]({{ site.baseurl }}{% post_url 2018-01-20-how-to-migrate-a-raspberry-pi-installation-to-lvm %}) - it wasn't. I also had to rebuild the `initramfs` boot image. That itself is fairly easy. Unfortunately - with some more files now on `/boot` - there wasn't enough space anymore.

From here, there are multiple ways to solve it. For me, the task was clear. I added a new flag [bcrm](https://github.com/jeansen/bcrm), cloned the version 3 SD card and increased the boot partition to 100 MB. Then I copied over the new files, and now all is well.

Let's see what this new version 4 is capable of ;-)


 
