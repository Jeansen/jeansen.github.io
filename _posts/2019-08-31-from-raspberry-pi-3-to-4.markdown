---
layout: post
title: From Raspberry Pi 3 to 4
tags: [raspberry pi, linux]
date: 2019-08-31T17:25:31+02:00
---

When I received the new Raspberry Pi 4 Model B (with 4GB RAM), I thought it would be as easy as taking the SD card from a Raspberry Pi 3 Model B+ and put it in the new version 4. Well, close but not exact. You will have to copy over some additional files for the boot partition. I managed this by simply taking the actual available Raspbian image, extracting it to a new SD card, booting it up (for verification) and then copying over the complete boot partition.

This should be enough in generally. Of course, in my case - with an unconventional [LVM setup]({{ site.baseurl }}{% post_url 2018-01-20-how-to-migrate-a-raspberry-pi-installation-to-lvm %}) - it wasn't. I also had to rebuild the `initramfs` boot image. That itself is fairly easy. Unfortunately - with some more files now on `/boot` - there wasn't enough space anymore.

From here, there are multiple ways to solve it. For me, the task was clear. I added a new flag [bcrm](https://github.com/jeansen/bcrm), cloned the version 3 SD card and increased the boot partition to 256 MB. That's the recommended size (otherwise rpi-update` will issue a warning). Then I copied over the new files, rebuild the initramfs file and now all is well.


## New Kernel

When you have copied over all files, you will see that there is a new Kernel file available. On my version 3 installation, `/boot` had two files:
    
    kernel7.img
    kernel.img

Now, there is a third one: 

    kernel7l.img
    
Also, if you check a clean new installation, you'll see that `/lib/modules` will have an additional folder, with the `l` suffix. For instance:

    4.19.57+  
    4.19.57-v7+  
    4.19.57-v7l+

`4.19.57-v7l+` is the kernel being used on version 4. Therefore the initramfs file has to be rebuilt, otherwise the boot process will fail because of inconsistent versions.

## What I did - some more details

So, to summarize: I took three SD cards. On the first (/dev/mmcblk0) I simply installed the latest Raspbian image and booted my new Pi 4 from it. The second (/dev/sda) had the version 3 installation on it and the third (/dev/sdb) was for the clone. After cloning, I mounted the boot partition (/dev/sdb1) and copied over all new files from `/boot` (/dev/mmcblk0p1). I then rebuilt initramfs:

    sudo mkinitramfs -o /boot/initramfs.gz 4.19.57-v7l+
    
Afterwards I put the cloned card in the new Raspbian Pi 4 and booted it up. All went fine. But the kernel used from the plain installation image was a bit old. In addition, the current installation was still missing the `-v7l+` module. For instance, my `/lib/modules` folder still had the following folders:

    4.19.66+  
    4.19.66-v7+  
        
Of course, I could have copied it over, but either way I prefer doing a clean upgrade. So, I checked `https://github.com/Hexxeh/rpi-firmware/commits/master` for the latest kernel. At the time of this writing, it was 4.19.69. The commit hash was `f8c5a8734cde51ab94e07c204c97563a65a68636`. So I executed:

    sudo rpi-update f8c5a873
    
You do not have to provide the full hash, the first 6 or more digits will do just fine. And again, I rebuilt (like always) the initramfs file:

    mkinitramfs -o /boot/initramfs.gz 4.19.69-v7l+

Another reboot, and I was ready to see what this new version 4 is capable of ;-)
