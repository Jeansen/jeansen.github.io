---
layout: post
title: Full disk encryption with LUKS, LVM and GRUB
date: 2018-06-30T00:29:34+02:00
tags: [linux, debian, lvm, grub, luks, encryption]
---

I have to admit. I am not an expert with LVM nor LUKS or GRUB. But this one brought me close to throwing in the towel!

Well, the idea was simple. Take a disk, encrypt it, and set up LVM on top of it. Of course, there are other ways of encryption, e.g. having an unencrypted boot partition or using encryption on top of LVM. But my goal was to have the encryption on the lowest layer and everything else on top, that is LVM on LUKS.

Initially I had hoped Veracrypt would to the trick but encrypting an already installed system is not supported for Linux as far as I can see and tell.

So, I searched the web, read blogs, articles and partly man pages. From what I found, there was no way around reinstalling my system. Too bad! Then I would have to backup my data first and restore it afterwards. Just the data, without the disk layout. Shouldn't be too hard ... 

Unfortunately all the examples either used 'initcpio' or 'dracut' for generating the initramfs image. These tools are not available on Debian. But this was the least problem. I just had to find a way to adapt some step(s).

Following is an effort of a whole day spent with countless tries. And even now it is more of a workaround than a solid solution!

But, let's get things ordered. Let me give you the details on my setup and how to get it encrypted. If you are not interested in some prose text, jump to the [technical details](#technial-playground).

### Starting point

Currently I have one disk with one partition that acts as a physical volume for LVM. This PV is a member of one volume group in which I have multiple logical volumes: One for swap, another for boot and yet another for my root filesystem. The remainder is unused. I use the remaining free space for snapshots which I then an when merge back in case one of my experiments failed ...

I created this layout for a reason. An now I would like to encrypt it. Completely!

By what I read, it should have been simple. Especially since GRUB for some time now understands encrypted devices and LVM.

### First steps

Filled with motivation to my fingertips I spun up my playground and followed the steps I found. I created a partition, encrypted it, set up a PV, VG and LVs. In addition I configured GRUB so it knows what to do and how to handle the encrypted disk. I edited '/etc/crypttab', '/etc/fstab' and '/etc/defaults/grub' accordingly and ran the necessary updates like 'update-initramfs', 'install-grub' and 'update-grub'. 

### The unexpected

Finally I started the new system. I got a prompt "ordering" me to type in the master key for decryption. So I did. Then, some seconds later, I saw the GRUB menu and then .... another prompt telling me to enter the decryption passphrase again.

So, the setup in general worked. I just had to type in the passphrase twice. I bet it could be done better. And indeed it should. But - indeed - it did not!

### The unexplained

What I understand is that GRUB was able to use my passphrase entered to decrypt the provided partition and to also mount all LVs. Otherwise I wouldn't have seen the GRUB menu. But when GRUB passes the control to the kernel, the encrypted drive is dismounted again.

The one "prominent" solution I found suggests to provide a keyfile in /etc/crypttab that will be used from within initramfs. So, one step is to create a keyfile. That is quite easy, by the way. The next step is to integrate this file somewhere in the initramfs image. That is a bit more work, but not complicated. To summarize, I had to write a simple hook script that would copy the keyfile into the initramfs image when calling 'update-initramfs'. With that file in place it then got interesting because I found half a dozen suggested configurations for /etc/crypttab or GRUB that all did not work (for me).

In the end, I wrote yet another script that I would call from /etc/crypttab to finally get things running. And actually, this is a [workaround](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=776409#74) for a [fixed bug from 2015](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=774647).

Scared? Don't be! What follows is a detailed walkthrough that will hopefully help you get started.


##Technial Playground

This is an advanced topic. So I assume you know how to setup a test environment and have some deeper Linux experience. 

I will use a Debian Live CD for this example. If you do not use Debian I suggest you first read some other articles. The guys from ArchLinux have som [very good documentation that you should read](https://wiki.archlinux.org/index.php/Dm-crypt).

I assume you will be root. Most of the following commands will need root permissions and therefore I spare myself the hassle of using sudo before almost every command.

With the live CD booted, here is my current disk situation:

    root@debian:/home/user# lsblk 
    NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    loop0    7:0    0    2G  1 loop /lib/live/mount/rootfs/filesystem.squashfs
    sda      8:0    0   88G  0 disk 
    sr0     11:0    1  2.2G  0 rom  /lib/live/mount/medium


As you can see, there is only one disk (sda) with 88G of free space. That is the disk we want to encrypt. Before we can do that, we will need some packages:

      apt update
      apt install lvm2 debootstrap cryptsetup

Now, we will setup the disk layout. 'cfdisk' is a great tool for this task because it is interactive and easy to use:

      cfdisk /dev/sda
            
When running cfdisk, choose 'dos' for the 'label type' and create one primary partition using all the available disk space.
      
Next, we will encrypt the disk:

      cryptsetup luksFormat /dev/sda1
      
To access the encrypted disk, we have to open (decrypt) it:

      cryptsetup open /dev/sda1 lukslvm --type luks
      
The last parameter is a name for the device mapper. After you have run the above command, you will have a device mapping of '/dev/mapper/lukslvm' available.


You should now have the following layout:
    
    root@debian:/home/user# lsblk 
    NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
    loop0         7:0    0    2G  1 loop  /lib/live/mount/rootfs/filesystem.squashfs
    sda           8:0    0   88G  0 disk  
    └─sda1        8:1    0   88G  0 part  
      └─lukslvm 254:0    0   88G  0 crypt 
    sr0          11:0    1  2.2G  0 rom   /lib/live/mount/medium

Now, we can create the LVM layer:

      pvcreate /dev/mapper/lukslvm
      vgcreate vg00 /dev/mapper/lukslvm
      lvcreate -l +100%FREE vg00 -n root
      mkfs.ext4 /dev/mapper/vg00-root
      
And here is the final layout with LVM on LUKS:

    root@debian:/home/user# lsblk 
    NAME            MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
    loop0             7:0    0    2G  1 loop  /lib/live/mount/rootfs/filesystem.squashfs
    sda               8:0    0   88G  0 disk  
    └─sda1            8:1    0   88G  0 part  
      └─lukslvm     254:0    0   88G  0 crypt 
        └─vg00-root 254:1    0   88G  0 lvm   
    sr0              11:0    1  2.2G  0 rom   /lib/live/mount/medium

Time to install a system:

      mkdir /mnt/root
      mount /dev/mapper/vg00-root /mnt/root      
      debootstrap --include linux-image-amd64,grub-pc,locales --arch amd64 stable /mnt/root

After that we use chroot for all further tasks:
      
      for f in sys dev dev/pts proc run; do mount --bind /$f /mnt/root/$f; done
      chroot /mnt/root
      
Then run 'dpkg-reconfigure locales' and select en_US.UTF-8 in all dialogs. Without this you will get some annoying warnings and some packages could fail installing. After that, install the remaining packages:
      
      apt install lvm2 cryptsetup keyutils
      
Create the file '/home/dummy' and add the following two lines:

    #!/bin/sh
    exec /bin/cat /${1}
    
Make sure this script is executable: 'chmod +x /home/dummy'

We also need a keyfile:

    dd bs=512 count=4 if=/dev/urandom of=/crypto_keyfile.bin
    cryptsetup luksAddKey /dev/sda1 /crypto_keyfile.bin
    chmod 000 /crypto_keyfile.bin
      
Now, we can edit '/etc/crypttab':

    echo "lukslvm /dev/sda1 /crypto_keyfile.bin luks,keyscript=/home/dummy" >>  /etc/crypttab
      
This file is similar to '/etc/fstab' and handles automatic decryption of filesystems. The line above would translate to "decrypt /dev/sda1 by calling /home/dummy /crypt_keyfile.bin".

Of course it is not enough to decrypt /dev/sda1. We also need to mount the root filesystem:

    echo "/dev/mapper/vg00-root   /       ext4    defaults        0       1" >> /etc/fstab

In addition, we need to configure GRUB and make it aware of the encrypted filesystem by editing '/etc/default/grub':

    GRUB_CMDLINE_LINUX="cryptdevice=/dev/sda1:lukslvm"
    GRUB_ENABLE_CRYPTODISK=y

Finally we have to make the script and keyfile defined in '/etc/crypttab' available during boot. We do so by adding them to the initramfs image. This requires a custom script that has to reside in '/etc/initramfs-tools/hooks'. The name does not matter. I called it 'lukslvm'. And here is its content:

    #!/bin/sh
    
    set -e
    
    PREREQ=""
    
    prereqs()
    {
            echo "$PREREQ"
    }
    
    case $1 in
    prereqs)
            prereqs
            exit 0
            ;;
    esac
    
    . /usr/share/initramfs-tools/hook-functions
    
    cp -a /crypto_keyfile.bin $DESTDIR/crypto_keyfile.bin
    mkdir -p $DESTDIR/home
    cp -a /home/dummy $DESTDIR/home
    
    exit 0
    
Do not forget to make it executable:

    chmod +x /etc/initramfs-tools/hooks/lukslvm

Scripts in '/etc/initramfs-tools/hooks/' are run in addition to scripts from '/usr/share/initramfs-tools/hooks/'. Have a look there for further impressions and check the man page (8) of initramfs-tools for more details.

Finally, we can install GRUB and update the initramfs image:

    grub-install /dev/sda
    update-grub
    update-initramfs -u
    
Make sure the "dummy" script and key file have been added to the initramfs image:

    lsinitramfs /boot/initrd.img-$(uname -r) | grep -E 'dummy|crypto_keyfile'
     chmod 600 /boot/initrd.img-$(uname -r)
  
And set the root passwort (if you want to be able to log in!)

    passwd
    # insert your new root password
    
That's about it. You should now be able to reboot (remove the Live CD!) and have a fully encrypted system with LVM on LUKS.

