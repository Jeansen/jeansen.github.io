---
layout: post
title: Emulate Raspberry PI with QEMU
tags: [raspberry pi, qemu, linux]
date: 2018-06-03T14:45:36+02:00
---

While developing software for the Raspberry Pi, it quickly became a pain to constantly have switch SD cards for testing purposes.

So I was looking for an option to emulate a Raspberry Pi locally which would allow me to speed up testing and have a faster feedback cycle. A nice side effect: I could tidy up my messy desk (a bit)!


# Before you start

Make sure you install the following packages:

    sudo apt install qemu-system-arm
    
You will need two thing: A Raspbian Lite image and a custom kernel for QEMU.

## Raspbian Lite

You can get Raspbian Lite from the [official site](https://www.raspberrypi.org/downloads/raspbian/).

## Kernel

As I learned, the standard kernel cannot be used together with QEMU. The community standard seems to be the kernels you can find in the
[dhruvvyas90/qemu-rpi-kernel](https://github.com/dhruvvyas90/qemu-rpi-kernel) repository. That's what worked for me, too. so, simply clone the repository:
  
    git clone https://github.com/dhruvvyas90/qemu-rpi-kernel.git

# Preparing the virtual disk

You should now have everything you need: QEMU, custom kernels and a Raspbian Lite image. But to use the Raspbian Lite image with QEMU, you will need to:

- extract the image from the downloaded zip file
- convert it to a more efficient type
- extend the disk
- extend the second partition and filesystem
- change some configuration files before first boot.

First, unzip the Raspbian Lite zip file you just downloaded:

    unzip 2018-04-18-raspbian-stretch-lite.zip
  
Converted it to a more native and flexible image type which allows [multiple snapshots and much more](https://en.wikibooks.org/wiki/QEMU/Images):

    qemu-img convert -f raw -O qcow2 2018-04-18-raspbian-stretch-lite.img 2018-04-18-raspbian-stretch-lite.qcow2
    
The "native" and  
    
Extended it by 6G (or whatever you like):

    qemu-img resize 2018-04-18-raspbian-stretch-lite.qcow2 +6G

To access the disk, the Network Block Device driver needs to be loaded:

    sudo modprobe nbd

Connect the image as a block device:

    sudo qemu-nbd -c /dev/nbd0 2018-04-18-raspbian-stretch-lite.qcow2 
    
Use `cfdisk` to delete and recreate the primary partition `/dev/sda2` :

    sudo cfdisk /dev/nbd0

Extend the file system:

    sudo e2fsck -f /dev/nbd0p2
    sudo resize2fs /dev/nbd0p2

Now, the [wiki from the custom kernel repository](https://github.com/dhruvvyas90/qemu-rpi-kernel/wiki) states, that you must edit 2 files. Therefore you will need to mount the second partition:

    sudo mkdir /mnt/nbd0p2
    sudo mount /dev/nbd0p2 /mnt/nbd0p2

Comment (with # symbol) the first (and only) line in `/mnt/nbd0p2/etc/ld.so.preload`:

Then create the file `/mnt/nbd0p2/etc/udev/rules.d/90-qemu.rules` and put the following lines in it:

    KERNEL=="sda", SYMLINK+="mmcblk0"
    KERNEL=="sda?", SYMLINK+="mmcblk0p%n"
    KERNEL=="sda2", SYMLINK+="root"

Finally do some cleanup:

    sudo umount /mnt/nbd0p2
    sudo qemu-nbd -d /dev/nbd0
    sudo rmmod nbd
    
# Booting QEMU with Raspbian Lite

The boot command for `qemu-system-arm` is rather complex but not complicated. You can take what you find on the [wiki](https://github.com/dhruvvyas90/qemu-rpi-kernel/wiki) and adapt the flags for `-kernel`, `-dtb` and `-hda` accordingly.

    qemu-system-arm \
    -kernel qemu-rpi-kernel/kernel-qemu-4.9.59-stretch \
    -cpu arm1176 \
    -m 256 \
    -M versatilepb \
    -dtb qemu-rpi-kernel/versatile-pb.dtb \
    -no-reboot \
    -append "root=/dev/sda2 panic=1 rootfstype=ext4 rw" \
    -net nic \
    -net user,hostfwd=tcp::5022-:22 \
    -hda 2018-04-18-raspbian-stretch-lite.qcow2 


## Set up SSH

Now, in your machine, you can run sudo raspi-config and enable the SSH server (in the "Interfacing Options" menu at time of writing).

In my experience, the emulation is OK, but could be faster. The resolution in QEMU is quite small, so I suggest you enable SSH in the "Interfacing Options" with `sudo raspi-config`. Now, you can connect into the QEMU with e.g. `pi@localhost -p5022`.
