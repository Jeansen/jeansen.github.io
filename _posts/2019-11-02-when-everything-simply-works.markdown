---
layout: post
title: When everything simply works ...
date: 2019-11-02T14:22:10+01:00
tags: [linux, bcrm, the real thing]
---

It's almost scary (no, not because it is Halloween time ). Today I had to clone my system because my boot partition got too small. And though I always put as much effort as possible in everything I do, I am always skeptical. Yes, I did a lot of testing, but still some doubts remained. After all, this time it was not for toying around but for real. If things went wrong, I'd be very ......... "upset"!

But well, no risk on fun, right? So, I put my beloved [bcrm project](https://github.com/Jeansen/bcrm) to the test. Frankly I do not like to praise myself, but this time; What should I say? It simply worked flawlessly!

## The real thing

The task I saw myself confronted with was simple: Clone from a Debian live CD, backup the current system and then restore it with some changes to the partition table. Bcrm is able to do all this for me. Here's the summary:

1) I downloaded the current [Debian Live CD (with nonfree firmware images)](https://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/), copied it to an USB stick and booted from it. I still like to use [rufus](https://github.com/pbatard/rufus) for this (needs Windows).
2) After having booted into the Live CD, I installed some needed packages (bcrm would tell me to install them otherwise) and cloned the [bcrm project](https://github.com/Jeansen/bcrm)
4) Since I use LVM, I had to scan for possible volume groups and activate the logical volumes (I already created an issue for this to make it transparent in bcrm)
5) Then I attached an external disk and ran bcrm:
    
        #Create backup
        ./bcrm.sh -s /dev/<source> -d /mount/of/external/disk/backup/folder
        
        #Restore from backup and change boot partition size
        ./bcrm.sh -d /dev/<source> -s /mount/of/external/disk/backup/folder -b 1G
        
