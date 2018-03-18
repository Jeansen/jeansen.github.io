---
layout: post
title: Moving to LVM
tags: []
date: 2016-07-01T12:54:24+02:00
---

For some time now I've been using Linux in a virtual Machine. The snaptshot feature provided by most hypervisors is perfect for experiments or just to make sure you have a state you can easily revert to, if some package installations go wrong or otherwise mess up your system. But what do you do in the absence of a virtual machine? Welcome to LVM!

Simply put LVM (Logical volume manager) introduces a layer of abstraction for mapping *mapping physical block devices onto higher-level virtual block devices.* [^1]

With this abstraction it is possible to group physical volumes (PV) in a so called volume group (VG). A volume group again can be devided in logical volumes (LV). What makes LVM so interesting are two things. On the one hand LVM offers the ability to extend a VG by simply adding another PV. If you need more storage space, just add another PV. Being able to extend a VG implies the possibility for dynamic resizing LVs. That is, if you added a PV to a VG and want to make this space available to one or more LVs you simply resize them - on the fly!

 On the other hand LVM offers a feature called snapshots. This feature is similar to snapshots offered by hypervisors, though a bit different. But the idea is the same. You create a snapshot of a given change, do some changes and if you do not like what has changed you simply revert the snapshot.


#Migrationg to LVM
With all the nice features provided by LVM I found myself in the situation that I did not setup my existing Linux installation with LVM when I first installed it. Since the installation a lot changed. So doing a complete reinstall without any configuration management just scared the hell out of me! So, the challenge was to migrate the existing installation to LVM. Here's how I accomplished that. To be on the save side I first ran all of this in a virtual machine.

__PLEASENOTE__
The following explanation assumes that there is only one root partition. /home, /usr, /boot etc reside on the same partition!

## 1. Preparations
- Get a LiveCD. For instance [Debian](https://www.debian.org/CD/live/).
- Create an additional hard disk and attach it to your existing virtual machine.
- Create a new virtual machine with the settings you want for your new machine.

Boot your current virtual machine with the LiveCD, format the additional disk and mount it. Doing this graphically can be done with `Gnome disks`. Otherwise `fdisk`, `mkfs` and `mount` are your friends to create partitions, format and mount them.

In this scenario I just need one partition to store my backup file. Actually I would not need a partition but we will need the command later on, so I will show it here.

To see what is available use `lsblk`. An example output might look like the following:

    NAME                        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda                           8:0    0   128G  0 disk
    ├─sda1                        8:1    0   250M  0 part /boot
    ├─sda2                        8:2    0     1K  0 part
    └─sda5                        8:5    0 127.8G  0 part
      ├─root-root               254:0    0 123.8G  0 lvm  /
      └─root-swap               254:1    0     4G  0 lvm  [SWAP]
    sdb                           8:16   0    80G  0 disk
    ├─sdb1                        8:17   0     1K  0 part
    └─sdb5                        8:21   0    80G  0 part

Creating a partition with `fdisk` is fairly easy. For instance, to create a partition on /dev/sdb you run `fdisk /dev/sdb` which brings up the interactive command shell of fdisk for manipulations on /dev/sdb. You can do whatever you like. Until you do not write your changes nothing will be altered physically. The command shell should be easy to get along with. After all, there are only about a dozen commands which all just have one letter ;-)

So, with fdisk running I first type `n` for a new and follow it with `p` to create a primary partition. I do not have special needs regarding the size, so I keep the defaults and just hit enter. With the primary partition created I hit `p` to see my changes so far and double check everything is according to my specifications. `w` finally writes the changes to the disk.

`lsblk` should now show you, that you have one partition `sdb1` under `sdb`. We can now format this partition with `mkfs.ext4 /dev/sdb1` and mount it, e.g. `mkdir -p /mnt/backup; mount /dev/sdb1 /mnt/backup`.

## 2. Backup your system
To backup my data I wanted to make sure most of the attributes stay as they are. There are a lot of different tools that can do the job. Since I did not want to do a sector-by-sector cloning I decided to use tar. Here's how I created my backup file: 

    tar -Scpf /mnt/backup/backup.tar --directory=/media/user/UUID/ --exclude=/proc/* --exclude=/dev/* --exclude=/sys/* --atime-preserve --numeric-owner --xattrs .

- **-S** This option instructs tar to test each file for sparseness before attempting to archive it. If the file is found to be sparse it is treated specially, thus allowing to decrease the amount of space used by its image in the archive[^2].

- **-p** This option causes tar to set the modes (access permissions) of extracted files exactly as recorded in the archive.[^3]
- **--atime-preserve** Preserve the access times of files that are read.[^3]
- **--numeric-owner** Create extracted files with the same ownership they have in the archive.[^3]
- **--xattrs** Enable extended attributes support.[^4]

- **--exclude** Causes tar to ignore files that match the pattern.[^5]
- **--directory** Changes the working directory in the middle of a command line. [^6]
- **-f** Name the archive to create or operate on.
- **-c** Create a new archive


`/proc, /sys, and /dev` are temporary file systems which get populated on boot. Therefore I do not include them in my backup because they might interfere with the normal population process (which can change on any upgrade).[^7]

I omitted `-v` because I do not like to clutter my terminal with uninteresting output. But you might want to add `-v` to the command above to see the actual files being archived.

After some time (maybe get a fresh coffee) tar finished successfully and my complete system got stored in a single file in /mnt/backup/backup.tar.

##3. Prepare LVM

##4. Restoration

Restoring my system consisted of three parts. First, I had to create my future system layout. Second, I had to restore my data and finally create a boot loader with GRUB and adapt `/etc/fstab`.

### System layout

### Restore files with tar

### Prepare boot

After tar finished extracting my backup files, there was still some work left because in the current stat I only had a disk with some date. Being able to boot into the new system still requied a boot loader like GRUB. 

For this I had to run my currently bare system in some sort of a minimal environment. There are multiple choices like `schroot`, `systemd-nspawn `or the good old `chroot`. I chose the last one but feel free to tinker with the others, if you like to.

Bevore I was able to run chroot I had to bind mount `dev`, `dev/pts`, `sys` and `proc`. Here is a little one-liner I used for this task.

  for f in dev dev/pts sys proc ; do mount --bind /$f /media/user/UUID/$f ; done
  
After that I could run chroot like so: `chroot /media/user/UUID` and (re)installed grub with: `dpkg-reconfigure grub-pc`

Finally I had to replace obsolete UUIDs in `/etc/fstab`. To get the new UUIDs is used `blkid`. 

Now, I had a disk with a fresh boot loader and all my data restored.

##5. Final words

Time to boot into the new system!




[^1]: https://en.wikipedia.org/wiki/LVM2
[^2]: http://www.gnu.org/software/tar/manual/html_node/sparse.html
[^3]: http://www.gnu.org/software/tar/manual/html_node/Attributes.html#SEC139
[^4]: http://www.gnu.org/software/tar/manual/html_node/Extended-File-Attributes.html#SEC70
[^5]: http://www.gnu.org/software/tar/manual/html_node/exclude.html#IDX393
[^6]: http://www.gnu.org/software/tar/manual/html_node/directory.html#IDX458
[^7]: https://wiki.archlinux.org/index.php/Full_System_Backup_with_tar