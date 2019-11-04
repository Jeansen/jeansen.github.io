---
layout: post
title: GRUB + LUKS2 - not just yet!
date: 2019-11-02T23:05:19+01:00
tags: [linux, bcrm, luks, grub]
---

So, while having been over-motivated by [my previous post]({{ site.baseurl }}{% post_url 2019-11-02-when-everything-simply-works %}), I went on and wanted to enhance the encryption part of [bcrm project](https://github.com/Jeansen/bcrm). Currently encryption only works with LVM and no other partitions in place. But in a lot of default setups `/boot` is on a separate partition and then there is the story of EFI with its own system partition.

So, before doing any new enhancements, I ran the current implementation but this time not with Debian Stretch, but the current stable release, Buster. Unfortunately the test failed. I could not figure out why until I learned that from Stretch to Buster the `crypt-setup` package bumped from version 1.x to 2.x reflecting the new LUKS2 format.

Unfortunately [GRUB does not (yet) support LUKS2.](https://savannah.gnu.org/bugs/?55093). So, for now you will need to run something like the following:

    cryptsetup luksFormat /dev/sda1 --type luks1
    
This seems to work with all versions of `cryptsetup`.




