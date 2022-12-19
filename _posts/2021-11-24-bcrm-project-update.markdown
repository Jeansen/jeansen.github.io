---
layout: post
title: BCRM project update
date: 2021-11-24T23:48:42+01:00
tags: [linux, bcrm, vagrant, virtualbox, kvm, libvirt]
---

# 2021-11-24
Because my "little" project [bcrm](https://github.com/Jeansen/bcrm) over the years grew bigger and bigger, I decided
about a year ago to implement automated tests.

Challenge accepted, I thought. But how would I best implement those tests? On the one hand, the complete project is an
overly complex shell script. On the other hand, I had to figure out a way to test the creation of a complete operating
system.

That's where Vagrant came into play. But this is only half of the rent. To utilize Vagrant, I needed some templates.
A companion tool to [Vagrant](https://www.vagrantup.com/) is Packer.  With [Packer](https://www.packer.io/) I was able
to create Vagrant boxes that I could provision, pack and reuse for every test. Actually, there is a lot more involved.
For instance, I also had to utilize Debian's installer with custom [preseed
files](https://wiki.debian.org/DebianInstaller/Preseed). But I won't go into details here. Let me just say this: It took
me many, many days of trial and error until I had a working set of Vagrant boxes. I've never raised and destroyed so
many virtual machines!

Next, I had to write tests. That's where I decided to use simple shell scripts again. Depending on the test, each stage
would set different environment variables and run different SSH calls against a provisioned Vagrant box.

I also had to learn some Ruby, because Vagrant files are written in Ruby.

Naturally, I did not want to run tests every day or every week by hand. I needed continuous integration. Therefore,
I set up an older Intel NUC computer with [Jenkins](https://www.jenkins.io/) and [Gitea](https://gitea.io), including
[Git LFS](https://git-lfs.github.com/). The latter I needed because I decided to use Git not only for the source code
involved, but also for the template boxes.  I could have uses public repositories, like Github. But pushing Gigabytes of
Gigabytes into public repositories is slow and expensive. 

Next, I had to configure Jenkins and write some Pipelines (in Groovy).

Finally, I had to write tests. Actually, it was more a matter of documenting the output of each manual run. That is,
I had to run every test I came up manually and take the output for future test runs. For example: Partition tables, LVM
tables, checksums, list of created backup files, etc. During this procedure, I also fixed some tricky bugs. That again
proves, how important tests are, no matter how complex.

## Going KVM

After about a year, I had all this set up (in my spare time). And it worked quite well. But one thing bothered me:
[Virtual Box](https://www.virtualbox.org/) is a level 2 hypervisor which technically makes it slow. For my purposes
a level 1 hypervisor would suffice and probably speed up execution times.

With the first iteration done, the next iteration was then to implement KVM support. Again, it involved a tremendous
amount of trial and error but ultimately I also had KVM support working and could then decide how to execute my tests:
either with a provider set to VirtualBox or [KVM/libvirt](https://libvirt.org/).

I also hacked together a script that would update all my boxes to a next Debian release, I even made Gitea call Jenkins
(by means of a web hook) and have the CI pipelines run whenever I would commit something to the test repository or the
Github repository of bcrm.

## Size matters

Today I upgraded all my boxes from Debian Buster (10) to Bullseye (11). With all the magic above in place,

But what totally struck me odd was the following:

    647M -rw-r--r-- 1 marcel marcel 647M 2021-11-24 libvirt-debian-bios.box
    654M -rw-r--r-- 1 marcel marcel 654M 2021-11-24 libvirt-debian-bios-lvm-all.box
    653M -rw-r--r-- 1 marcel marcel 653M 2021-11-24 libvirt-debian-bios-lvm-all-in-one.box
    650M -rw-r--r-- 1 marcel marcel 650M 2021-11-24 libvirt-debian-bios-lvm.box
    654M -rw-r--r-- 1 marcel marcel 654M 2021-11-24 libvirt-debian-efi.box
    656M -rw-r--r-- 1 marcel marcel 656M 2021-11-24 libvirt-debian-efi-lvm-all.box
    659M -rw-r--r-- 1 marcel marcel 659M 2021-11-24 libvirt-debian-efi-lvm-all-in-one.box
    657M -rw-r--r-- 1 marcel marcel 657M 2021-11-24 libvirt-debian-efi-lvm.box
    1.9G -rw-r--r-- 1 marcel marcel 1.9G 2021-11-24 virtualbox-debian-bios.box
    1.9G -rw-r--r-- 1 marcel marcel 1.9G 2021-11-24 virtualbox-debian-bios-lvm-all.box
    1.6G -rw-r--r-- 1 marcel marcel 1.6G 2021-11-24 virtualbox-debian-bios-lvm-all-in-one.box
    1.8G -rw-r--r-- 1 marcel marcel 1.8G 2021-11-24 virtualbox-debian-bios-lvm.box
    1.9G -rw-r--r-- 1 marcel marcel 1.9G 2021-11-24 virtualbox-debian-efi.box
    2.3G -rw-r--r-- 1 marcel marcel 2.3G 2021-11-24 virtualbox-debian-efi-lvm-all.box
    2.3G -rw-r--r-- 1 marcel marcel 2.3G 2021-11-24 virtualbox-debian-efi-lvm-all-in-one.box
    2.0G -rw-r--r-- 1 marcel marcel 2.0G 2021-11-24 virtualbox-debian-efi-lvm.box

As you can see, boxes for libvirt are round-about a third the size compared to boxes packed for VirtualBox. Execution
time is also dropped by half.  With VirtualBox every test stage took about 30 minutes. With KVM/libvirt, it only took
about 15 minutes! That is a lot, especially when a complete test suite takes about 2 days to finish!

It is nice to have a choice. But I think I will soon drop VirtualBox for my tests.

## ROI

I learned a lot during the implementation. And I have to admit that I caught myself more than once thinking about the
effort and if it's worth it. But the more features I added to bcrm, the more I found myself running tests. At some
point testing took more time than implementing a feature. That's where I decided I had to implement automated tests. And
yes, it took **a lot** of time. But this time was well spent. Not only did I find some nasty bugs during the
implementation and setup. I now have only to worry about new tests for new features. Whenever I fix a bug and commit the
change, a full machinery of tests is triggered.

I can now fully concentrate on new features (and some additional tests). What remains is free time. And if a test fails,
then it is either because I introduced a bug (maybe a regression) or because of other (external) changes that need
attention. For instance, I had one case where the fdisk tool got updated by a minor version but the dump output changed.
All my tests failed because every assertion was missing a newly introduces 'sector' field.

## Conclusion
Tests are time-consuming. Especially system tests. But either way, the investment pays off. Depending on the test
environment you might have to utilize a dozen tools for integration and automation. This can be challenging and even
make you step back and drop the whole idea. Fortunately with private projects, it is all about fun and education. And
looking back I can only confirm this. The journey is the reward! And there are even more journeys to come!

# 2022-19-12
One year passed since my last posting regarding [bcrm](https://github.com/Jeansen/bcrm). And I am very pleased that 
with more and more automated test I found different bugs I would otherwise have not spotted. But writing these kind of
system tests is - simply put - hard! But not harder than fixing some bugs.

## Device and conquer
One prominent algorithm, when it comes to sorting, is 'devide and conquer'. And one could claim I do exactly this in
bcrm. Of course, in a very simplified way. The process of cloning can be broken down in two phases. First I collect all
the date from the source device and sort it. Then I prepare the destination device and create the initial structure
followed by a similar collection of all the date from the destination device, in a sorted manner. With these two lists
I have a one-to-one mapping. Each part (folder, partition,file) from the source side is mapped to a corresponding part
(folder, partition,file) on the destination side.

So far the theory. As it turns out, getting thins ordered the right way and uniquely mapped is another discipline. When
I started implementing bcrm, I was meant to be a simple (and small) helper script. Now, it already counts more than 3600
LOC. It's not the most beautiful code and some parts are pure "code golf", but it does the job. But that is also on
reason why I did not decide to use more unique mounting handlers and other data structures. So, for the moment, I rely
on `lsblk`. But depending on the output formatting, the order also changes. Admittedly, I have not figured it out
completely. Using raw mode (but the same columns) produces a different output than list mode. And again, the output
changes with different selection of columns. It took me quite some time to figure it out and find a uniform and reliable
selection.


# All tests green
But now, all tests are green again. And there are a lot of tests by now. Running on a dedicated Intel NUC all tests take
about 3 days. Tests are dived in suites, stages and pure tests. For instance, there is a test suite taking a system with
one LVM partition and the others plain. This suite runs all tests twice. One time against an EFI setup and aanother time
against a BIOS setup. Next, there are three stages for each test. The first stage clones a running system live. The
second stage simulates the use-case where a system is not cloned live but from a LiveCD or another system. That is, the
system to be cloned is on an external disk or the system to be cloned was booted from a LiveCD. The third and last stage
clones a system to virtual images. Each stage then is again composed of multiple test. One test simply clones a system.
Another test creates a backup. Yet another test restores the backup from before. And yet another test does the same, but
clones into virtual images, e.g. VDI or QCOW2. After that each clone is booted, selected configurations are exported
(e.g. partition tables) and compared against assert files. 

Currently, this makes about 4 suites (counted twice), 8 stages, 5 tests eacht: 4 * 2 * 8 * 5 == 320 tests. Each suite
consumes about 8 hours. So, yes. Running all tests takes bout  64 hours or roughly 3 days. These tests only cover the
most fundamental cloning scenarios. If I were to include all possible tests, I bet it would take a month or more for one
complete run.


## What's next?!
That being said, I will add more tests, anyway. Covering 80 percent should suffice. Edge cases will be added, as needed.
For the moment I have other "problems" to solve. I added some more ideas and the project board is still well filled for
new versions to come. I wonder when I will raise the version to '0.2'. Next year, I definitely plan to implement
incremental backups and refactor the code for Bash 5.0 script features.  








