---
layout: post
title: A DevOp's journey
date: 2020-04-05T16:13:53+02:00
tags: [linux, devops, vagrant, jenkins, debian]
---


Normally I develop software. That's what I like, being creative and building things. But, every good peace of software needs to be tested. Depending on the technology, one has multiple tools or frameworks at hand.

But how do you test system installations? Of course, you can do it manually, but how would automate it?

These were upcoming questions with one of my projects. Whenever I fixed an issue or implemented a feature, I ran some manual tests in a couple of virtual machines (which I had created manually, as well).

But the more features I had implemented, the bigger the test matrix became. And with all the manual testing, the more time I had to invest.

So, I asked myself, how could I automate the whole process? That is, I wanted to have the following in place:

- Predefined virtual images that I could easily create and reuse.
- Automated, deterministic tests
- Test environment that would run whenever something was checked in to git.

Now, after some weeks of fun (no further comment here), I now have a working setup which includes:

- Debian preeseeding to create definitions for automated system installations and provisioning
- Packer for building System images (to be used with Vagrant)
- Vagrant for running tests in deterministic and clean environments
- Shell scripts to be used for automatic testing and image creation
- Jenkins with different jobs for automated building and testing


## Debian Preeseeding

Preseeding is used by the Debian installer to run predefined installations. Though the configuration isn't that complicated. The trick is to find the right settings, because there are so many of them. But they are quite self-explanatory and well documented. On the other hand, creating predefined disk setups including the partition table, filesystem and LVM took some time. Even now, it is sometimes more a guess. But, it works and the outcome is as expected.

## Packer and Vagrant

I use packer (as recommend by the Vagrant project) to create base images. In essence my packer configuration takes an ISO file, boots up a virtual machine and kicks off the installation process. The latter one is then what the preeseeding files will be used for.

Vagrant can then be used to import a base image and use that base image as a starting point for new virtual machines. This way, it is a lot easier and faster to create a new virtual machine and configure it as needed. You could compare it to cloning an existing virtual machine and tweak it for your current application by installing additional packages or simply configuring the VM with different settings (RAM, CPUs, etc.).

## Test setup

For my test setup I created a little shell script. That script will also be run by Jenkins.

## Jenkins

Jenkins is a tool for continuous integration and deployment. It is a very powerful tool, and I am not going to dive into it here.

## Virtualization on stereoids

Since VirtualBox version 6.1 nested virtualization is supported not only for CPUs from AMD but also from Intel. That made me think if it is possible to first create my build server with the right setup in a virtual machine and later on clone it to a real system.

Having a virtual machine is always a good idea when doing things for the first time. If you break something, you can easily restore a previous snapshot and try again. The only problem in my current setup was, that I actually needed to run a hypervisor in a hypervisor.

But, that's exactly what is possible with nested virtualization. And to my surprise it works quite well. Performance is a bit behind a real system, but still acceptable. I only had to make sure that any virtual machine has only one CPU cure. That is, even when the "outer" VM may have all CPU cores assigned (let's say 4), the "inner" VM can only have one! If you assign more than one CPU core, the outer VM or even the host might freeze. At least, that's what happend to me the first times I tried.

## Conclusion

Creating automated tests with Jenkins, Vagrant and predefined Images utilizing the Debian installer glued together with some shell scripting is complex but not complicated. And once it is all installed and configured, one can utilize it over and over again. That's the beauty of CI.
