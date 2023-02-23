---
layout: post
title: Synchronize time across Kubernetes nodes
date: 2023-02-23T01:34:26+01:00
tags: [kubernetes, linux, time, chrony]
---

Synchronizing time between virtual machines which do not have access to a hardware clock or time marker is a challenge. Especially, if these virtual machines serve as Kubernetes nodes and do not run all the time (paused, suspended). But, I found a working solution for me.

## Setup

In my case, I have several virtual machines setup with Vagrant and libvirt. These are provisioned as Kubernetes nodes and I bent them to my liking. Often, I create several snapshots, play around with new deployments and then revert to the latest snapshot, or an even older one.

Problem was, that after reverting to a snapshot from a couple of hours or days before, system time did not match. Sadly, even NTP clients like systemd-timesyncd or [chrony](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html-single/system_administrators_guide/index#ch-Configuring_NTP_Using_the_chrony_Suite
) did not work out of the box. I installed the later one because it allows for manual time synchronizations. But even then, it happened that host time and guest time diverged by several minutes.

Luckily, I found a working setup in RedHat's [KVM Guest Timing Management](https://access.redhat.com/documentation/de-de/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/chap-kvm_guest_timing_management).

    #As root execute the following commands in any guest VM
    apt install chrony -y
    modprobe ptp_kvm
    echo ptp_kvm > /etc/modules-load.d/ptp_kvm.conf
    echo "refclock PHC /dev/ptp0 poll 2" > /etc/chrony/conf.d/ptp_kvm.conf
    systemct restart chrony.service

Check that everything is right:

    sudo chronyc sources
    
There should be an Entry `#* PHC0`. Below is an example from one of my virtual machines.

    MS Name/IP address         Stratum Poll Reach LastRx Last sample
    ===============================================================================
    #* PHC0                          0   2   377     4    -29ns[  -87ns] +/-  148ns
    ^- static-186-155-28-147.st>     3  10   377     6  -1967us[-1967us] +/-  137ms
    ^- time.nullroutenetworks.c>     2  10   377   294  -2675us[-2617us] +/-   99ms
    ^- ns1.thorcom.net               1  10   377   185  -1637us[-1617us] +/-   20ms
    ^- time.cloudflare.com           3  10   377   703  +1702us[+1610us] +/-   15ms
    ^- fritz.box                     4  10   377   969  +1376us[+1068us] +/-   12ms

With that in place, the only thing you have to do is to call `sudo chronyc makestep` every time you need to align host and guest clocks. You could also just wait for some time (minutes, hours ...) until chrony catches up with constant time syncs.

Looping through running VMs and executing a 'makestep' is not elegant, but not more than a on-liner. With respect to my own CLI, which I wrote to better handle my cluster, I even added it to my 'revert' call. Whenever I call `k8s cluster revert`, a 'makestep' will be executed on every node.

## Time Travel

So, why is it actually so important to have all clocks synchronzied? Two practical examples. The first is curl. Whenever I reverted to a snapshot older than 2-3 hours (maybe even several days), some curl calls complained that x509 certificates were not yet valid. SSL handshakes simply failed because my nodes were too far behind.

Another example is Rook Ceph. Whenever I created a deployment with persistent storage, automatic bindings for a PVC stayed in 'Pending' mode. Looking at the logs, there were a lot of 'stale' entries and Rook complained of too less storage available. 

Time is critical in distributed storage systems and one is well-advised to have all connected systems synchronized as accurately as possible.

