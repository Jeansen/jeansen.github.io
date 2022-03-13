---
layout: post
title: Replacing Weave-Net with Flannel
date: 2022-03-13T18:06:46+01:00
tags: [kubernetes]
---

Since version 1.23 Weave-Net no longer works with Kubernetes. There has not been any update to the project for over a year and probably there will be no further updates. So, I replaced it with Flannel.

That was actually quite simple. I only hat do add the correct subnet and replace the weave-net YAML with Flannel.

And, you bet. There still was a minor glitch. After I had rebuilt cluster, my core-dns pods showed me something like this:

    ... network: failed to delegate add: failed to set bridge addr: "cni0" already has an IP address different from 10.244.0.1/24

For some reason 'cni0' NIC had a different IP than declared in `/run/flannel/subnet.env'

The solution was to simply delete the 'cni0' interface. It will be reassigned automatically:

    sudo ifconfig cni0 down
    sudo ip link delete cni0

Happy sk8ting ;-)
