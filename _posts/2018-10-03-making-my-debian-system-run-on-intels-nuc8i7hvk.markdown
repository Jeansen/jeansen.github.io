---
layout: post
title: Migrating my Debian system to Intel's Skull Canyon NUC8i7HVK
date: 2018-10-03T19:01:33+02:00
tags: [linux, debian, lvm, grub, uefi, amdgpu]
---

So, I finally decided to replace my old 5th generation NUC with a new one. And not any NUC, but the new Skull Canyon NUC8i7HVK. Because it took me a while to figure it all out, I want to share my knowledge with the community.

# The two showstoppers

So, I got the box, a new M.2 disk and 16 GB of DDR4 RAM modules. I cloned my current system over to the new disk using [my clone script](https://github.com/jeansen/pi2clone), installed the disk and RAM and fired up the box. Should have been that easy, right? Well, not exactly. Because of two reasons my system did not boot. 

The first reason was that Intel decided with the first Bios Update since the introduction of the Skull Canyon to exterminate the option for a legacy boot. That is, there is no option to disable UEFI anymore. As a result, you'll have to have a disk setup for UEFI. I did not, so I had to migrate (which I will explain shortly).

After I migrated my system to UEFI, I successfully got GRUB to show up and have the system boot up to the point where the boot log showed `fb: switching to amdgpudrmfb from efi vga`. There it hang. I could do a soft reboot with Ctrl-Alt-Del, but that's about it. Whenever I came to the above point, the system hang and would not go any further. One option to get pass that was to edit GRUB on the fly and add `nomodeset`. And after that the X server did not start, anyway. But, with `nomodeset` I got at working tty and was able to investigate some log files, eventually.

So, I checked the `~/.local/share/xorg/Xorg.0.log` log and found something like this: `Fatal server error: (EE) cannot run in framebuffer mode. Please specify busIDs for all framebuffer devices`. And in `/var/log/kern.log` I found some error entries saying that the firmware for something related to the AMD Radeon GPU did not load correctly (return code -2). As it turnes out I only had to install some more recent firmware files found here [https://people.freedesktop.org/~agd5f/radeon_ucode/vegam/](https://people.freedesktop.org/~agd5f/radeon_ucode/vegam/). Even so, I had no problems installing Debian stable (version 9.5) or booting from the Live CD.
 
 
# Setup the new (old) system
  
Now, I'll show in detail what I have done regarding the two issues above. Especially the migration to UEFI is one of many solutions. But one that worked for me :-)

Instead of converting my current system (clone), I took another approach and installed a fresh and clean Debian. For the installation I chose LVM and to have everything in one partition (the default for new users). I [cloned](https://github.com/jeansen/pi2clone) my current system again, but to an external USB drive. After that I booted from the Debian Live CD, wiped the root partition of the fresh installation, copied over the root partition from the clone and reconfigured GRUB. Here are the steps:

Running from the Live CD, I first had to install support for LVM.

    sudo apt-get update
    sudo apt-get install lvm2
  
After that I had to initialize the LVM.

    sudo vgscan
    sudo vgchange -ay my-src-vg
    sudo vgchange -ay my-dest-vg
    

There are two disks. One (sda) is the external disk with the clone. The other (nvme0n1) is the internal M.2 disk with the fresh Debian installation. Disk sda has two LVM volumes of which only the root partition was of interest to me. Disk nvme0n1 has three partitions:
  
  - nvme0n1p1 - EFI partition
  - nvme0n1p2 - boot partition
  - nvme0n1p3 - partition acting as PV for LVM
  
I then created two mount points and mounted the source and destination root folders.

    sudo mkdir /mnt/root_dest
    sudo mkdir /mnt/root_src
    sudo mount /dev/mapper/my-src-vg-root /mnt/root_src
    sudo mount /dev/mapper/my-dest-vg-root /mnt/root_dest
  
Next, I saved the current fstab file.

    cp /mnt/root_dest/etc/fstab .
  
Then I removed all the files from the destination.

    sudo rm -rf /mnt/root_dest/*
  
After that I synced the source with the destination.

    rsync -aSXxH /mnt/root_src/ /mnt/root_dest
  
Finally, I copied back the fstab file from before.

    sudo cp fstab /mnt/root_dest/etc/fstab
  
Next, I installed the new firmware files.

    git clone https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git
    sudo cp linux-firmware/amdgpu/vegam_*.bin /mnt/root_dest/lib/firmware/amdgpu
  
Then it was time to migrate the boot files. First I had to mount the destinations boot partition. Then I had to remove obsolete kernel files and copy over the new files.

    sudo rm -rf /mnt/root_dest/boot
    sudo mount /dev/sdb2 /mnt/root_dest/boot
    
    sudo rm /mnt/root_dest/boot/config-*
    sudo rm /mnt/root_dest/boot/initrd.img-*
    sudo rm /mnt/root_dest/boot/System.map-*
    sudo rm /mnt/root_dest/boot/vmlinuz-*
    
    sudo cp /mnt/root_src/boot/config-* /mnt/root_dest/boot
    sudo cp /mnt/root_src/boot/initrd.img-* /mnt/root_dest/boot
    sudo cp /mnt/root_src/boot/System.map-* /mnt/root_dest/boot
    sudo cp /mnt/root_src/boot/vmlinuz-* /mnt/root_dest/boot

Almost done. I only had to mount the efi partition, update the initramfs and update GRUB.

    sudo mount /dev/sdb1 /mnt/root_dest/boot/efi
    for f in sys dev dev/pts proc run; do mount --bind "/$f" "/mnt/root_dest/$f"; done
    chroot /mnt/root_dest
    
    #Now, while in chrrot    
    update-initramfs -u -k all
    update-grub
    
    
## Reboot!

So, I successfully copied over my system (root partition), installed new firmware files and updated the bootstrap files. Good to go!

# NOTE

I have deactivated "Secure Boot" in the BIOS and set iGD (found under Performance -> Graphics) to "Auto".