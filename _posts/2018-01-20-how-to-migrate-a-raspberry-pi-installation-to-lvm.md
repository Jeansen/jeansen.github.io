---
layout: post
title: How to migrate a Raspberry Pi installation to LVM
tags: [raspberry pi, linux]
date: 2018-01-20T15:27:08+01:00
---

In this post I will show you how you can migrate an existing Raspberry Pi installation to LVM.

The migration process can be summarized as follows:
- Backup you root partition
- Setup LVM
- Restore backup
- Adapt boot configurations


## Preconditions

Before you begin, make sure the installation you want to migrate has the `lvm2` package installed. Otherwise you will not be
able to boot after the migration!

## Hardware Setup
For this post I used a Rapsberry Pi 3 with an USB card reader and a 16GB SD card. Here is what my setup looks like:

    NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sda           8:0    1 14.9G  0 disk 
    ├─sda1        8:1    1 41.5M  0 part 
    └─sda2        8:2    1 14.8G  0 part 
    mmcblk0     179:0    0 14.9G  0 disk 
    ├─mmcblk0p1 179:1    0 41.5M  0 part /boot
    └─mmcblk0p2 179:2    0 14.8G  0 part /

In this example `mmcblk0` is the system disk and `sda` is the disk we will convert to use LVM.

__Make sure there are no mounts points on 'sda'!__


## IMPORTANT

__Please resist the urge to use `rpi-update` for kernel updates. This will break LVM!__


## Backup

By default, there are 2 partitions. One for `/boot` and the other for `/`. We we will not alter 'boot'. So we can leave 
it alone and backup the root partition `/` only. I will use [bcrm](https://github.com/jeansen/bcrm) for this purpose, 
but any backup with tar will suffice.

With bcrm it is as simple as providing the source disk and a destination folder. For this example we will
use `/tmp` as our workspace. So, let's get started and create a backup:

    cd /tmp
    mkdir /tmp/backup
    git clone https://github.com/Jeansen/bcrm.git 
    cd bcrm
    ./bcrm.sh -s /dev/sda -d /tmp/backup -i
    
## Setup LVM

With a backup in place, we can now convert the root partition to a Physical Volume (PV).

    pvcreate -y /dev/sda2
    
Then we need to setup a Volume Group (VG) which uses the previously created PV. For this example we will use the name 
'vg00'.

    vgcreate vg00 /dev/sda2
    
Finally, we can create one or more Logical Volumes (LV). In this example we will simply use all of the available space.

    lvcreate -n root -l100%FREE vg00
    
The commands 'pvs', 'vgs' and 'lvs' will give us a quick summary of what we have created. Let's see, what we have:
    
    root@raspberrypi:/tmp/bcrm# pvs
      PV         VG   Fmt  Attr PSize  PFree
      /dev/sda2  vg00 lvm2 a--  14.79g    0 
    
    root@raspberrypi:/tmp/bcrm# vgs
      VG   #PV #LV #SN Attr   VSize  VFree
      vg00   1   1   0 wz--n- 14.79g    0 
      
    root@raspberrypi:/tmp/bcrm# lvs
      LV   VG   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
      root vg00 -wi-a----- 14.79g 
      

      
Finally we need to create a filesystem:

    mkfs.ext4 /dev/mapper/vg00-root

## Restoring root

With LVM in place, we first need to mount the newly created LV.

    mkdir /media/root
    mount /dev/mapper/vg00-root /media/root
    
Although we have a full system backup, we only need to restore the root partition. Looking into the backup folder `/tmp/backup` 
reveals multiple files:


We need to extract the file that has `sda2`in its name:

    tar xf /tmp/backup/*_sda2* -C /media/root
    
## Adapt boot configuration

Since we now use LVM, we need a boot loader that can handle LVM volumes. Therefore we will need to create an 'initramfs'
image and tell the system where to find it.

Let's first mount the boot partition and add an 'initramfs' image.

    mkdir /media/boot
    mount /dev/sda1 /media/boot
    mkinitramfs -o /media/boot/initramfs.gz
    
Now, we need to tell the system where to find it:

    echo 'initramfs initramfs.gz followkernel' >> /media/boot/config.txt

    
In addition we will need to edit the kernel command line and filesystem table. 

First we replace the value of `root` in `/media/boot/cmdline.txt` with `/dev/mapper/vg00-root`. Here is an example
of what the final result could look like:

    dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=/dev/mapper/vg00-root rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait

In the same way, we have to change `/media/root/etc/fstab` for `/`. Here is an example:

    proc            /proc           proc    defaults          0       0
    PARTUUID=36c26eb6-01  /boot           vfat    defaults          0       2
    /dev/mapper/vg00-root  /               ext4    defaults,noatime  0       1
    
## Be prepared for new kernels

Finally, you'll have to make sure `initramfs` is rebuild whenever there are kernel updates. One options is to always 
do this after any APT upgrade sequence (done automatically by `unattended-upgrades`).

To have APT rebuild `initramfs` after every upgrade, you'll have to create a configuration file for APT, and a little
 script to select the right kernel version for `initramfs`.
 
Put the following in `/etc/apt/apt.con.d/30initramfs`

    DPkg::Post-Invoke {"/usr/local/bin/update_initramfs"}
    
Then create the file `/usr/local/bin/update_initramfs` with the following content and make it executable:

    #!/usr/bin/env bash
    cmd="mkinitramfs -o /boot/initramfs.gz $(ls /lib/modules | grep $(uname -r | grep -o 'v7.*') | sort -V | tail -n 1)"
    echo "$cmd"
    $cmd || exit 1
    exit 0
    
The catch here is to tell `mkinitramfs` to build for the new kernel and not the current one. This is necessary because after any kernel update the old one will be gone and `initramfs` would complain and fail because of missing folders in `/lib/modules`. This should be done without any user intervention, of course. Fortunately this is an easy task. With `ls /lib/modules | grep $(uname -r | grep -o 'v7.*') | sort -V | tail -n 1`you can select the latest "-v7" version and provide this to `mkinitramfs`.

### Why 30initramfs?
You might have to change the number, but 30 in `30initramfs` should be fine. This way we make sure the `Post-Invoke` hook is present before other tools like `unattended-upgrades` which by default use 50 or greater numbers. Also, note that the name `initramfs` as part of the filename is my personal preference.

### What happens if I dont' care for the kernel?

If you do not have initramfs rebuilt and you get a new kernel, then the new kernel will not be able to initiate LVM and
mount your filesystems. You might see something similar to the following and be stuck in emergency mode:

    WARNING: Failed to connect to lvmetad. Falling back to device scanning.
    /dev/mapper/vg00-root: open failed: Permission denied
    Failure to communicate with kernel device-mapper driver.
    Incompatible libdevmapper 1.02.145 (2017-11-03) and kernel driver (unknown version).
        
## All done

That's it! Put this newly created SD card in your Raspberry Pi and see what will happen :-) 
