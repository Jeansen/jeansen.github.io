---
layout: post
title: Speed up shutdown and boot times in Linux
tags: [linux, grub, systemd, lvm]
date: 2018-05-26T22:06:49+02:00
---

Usually, I do not restart my system until I have to because of some newly installed kernel. But recently I had to do some multiple reboots in a row and I wondered: Why does it take so long to shut down and boot?

I did not do any analysis for the shutdown problem. I just installed `watchdog` and that solved my problem.

For the exceeding boot times I ran `systemd-analyze time` and found two major time consumers. The first was `NetworkManager-wait-online.service`. I do not have to wait for the network to be online. So, I disabled it with `sudo systemctl disable NetworkManager-wait-online.service`.

The second one was `plymouth-quit-wait.service`. For this I had to edit `/etc/default/grub` and change the line `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"` to `GRUB_CMDLINE_LINUX_DEFAULT="noplymouth"` and afterwards did `sudo update-grub`.

If you use LVM and have any snapshots, you should delete or merge them before running `update-grub`. In addition make sure you do not have any external USB disks mounted (attached). Otherwise you might get errors like `grub-probe: error: unknown filesystem`.

If you get an error like `failed to remove /var/lib/os-prober/mount device or resource busy`, then just do `sudo umount /var/lib/os-prober/mount` and try again.

Finally, I saw a lot of `Running /scripts/local-block .. done.` lines polluting the console for about 10 to 15 seconds while booting. 

I first thought this was caused by some remains when I ported over my system from another installation and I simply had to replacing the old UUID in `/etc/initramfs-tools/conf.d/resume` with the current UUID of my current swap partition. Unfortunately this did not help. Clearing the `resume` file did not help, either.

Then I checked the man page for `initramfs.conf`, which states:

    VARIABLES FOR LOCAL BOOT
            RESUME
                  Specifies the device used for suspend-to-disk (hibernation), which the initramfs code should attempt to resume from.  If this is not defined or is set to auto, mkinitramfs will automatically select the largest available swap partition.  Set it to none to disable resume from disk.

So, I set `RESUME=auto`. But still, I got a bunch of these `Running /scripts/local-block .. done.` messages, having a massive impact on the system boot time. 

Finally, I set `RESUME=none` and that did the trick.
