---
layout: post
title: How to add kernel modules
date: 2018-05-19T22:45:11+02:00
tags: [Debian, Linux]
---

I guess it is the same with most of us when you have to fiddle around with some hardware settings then and when. You do
it once and then for months you no longer have to do anything until another challenge pops up. Well, today was such
a day for me.

I finally found some time to figure out how to get my wireless Logitech keyboard working while booting.

To have your hardware - in this case my keyboard - as soon available as possible, one can simply load modules into tne
initramfs.

For Debian based systems all available modules are stored in `/lib/modules/$(uname -r)/kernel/drivers/`. In this special
case I needed the module `/lib/modules/$(uname -r)/kernel/drivers/hid/hid-logitech-hidpp.ko`.

To have it loaded at boot time, I had to add the module name (without the extnsions .ko) to
`/etc/initramfs-tools/modules`. 

Finally I had to update the initramfs with `update-initramfs -u`.

And that`s it!

NOTE: Should you get some warnings of missing firmwares, [see my other post](how-to-fix-possible-missing-firmware-warnings.md).
