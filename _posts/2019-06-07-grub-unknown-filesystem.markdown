---
layout: post
title: Grub: Unknown filesystem
date: 2019-06-07T11:35:10+02:00
tags: [linux, grub, lvm]
---

This post will be short but hopefully be of help to you anyway!

Today I made some changes to my GRUB configuration. But when I ran `update-grub`, I found something like this in the output:

    grub-probe: error: unknown filesystem.
    /usr/bin/grub-probe: error: unknown filesystem.
    
The interweb is filled with explanations of what the cause might be. For me the reason simply was that I had - like always - an LVM snapshot in effect. Removing the snapshot resolved the issue!
    
