---
layout: post
title: Create a local kubernetes cluster with Vagrant
date: 2020-05-22T15:23:02+02:00
tags: [linux, kubernetes, docker, vagrant]
---

A couple of days ago I decided to bring myself up to speed again with kubernetes. But for a playground I needed a local cluster that I could break without harm or cost. Of course, there is `minikube` but it is rather limited. It is fine for a start, but when you want to have more control of resources, e.g. the number of nodes or the type of load balancers, you'll quickly reach the limits of `minikube`.

Fortunately I had been working on some other private project that already involved Vagrant. This way, it was quite easy to utilize my knowledge and hack together a script what would create a custom cluster for me.

For plain kubernetes, there is actually not much to it. The interesting parts then were how to install a load balancer with Ingress and being able to create dynamic Physical Volume Claims.

But after some trial and error and with the help of multiple blog resources, I finally had a little nice Vagrant script (and a very tiny shell script wrapper) that would do all the heave lifting for me.

You can find the [project on github with](https://github.com/Jeansen/k8s-playground).

