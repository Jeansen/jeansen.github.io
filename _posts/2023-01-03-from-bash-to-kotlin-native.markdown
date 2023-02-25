---
layout: post
title: From Bash to Kotlin Native
date: 2023-01-03T22:39:14+01:00
tags: [kotlin, linux, bash]
---

During the transition from 2022 to 2023, I had some days off and took that time between Christmas and the rest of the year to get my hands (finally) glowing again while hacking with Kotlin against POSIX and creating native executables. But, let's step back for a moment.

## A bit of history

Yes, I know. History is dry. I'll be quick. But I think it cannot harm to shed some light on the context and my thoughts behind the idea of writing something in Kotlin Native.

When I started with Kubernetes, especially during exam preparations, I wanted to have a cluster that I could bend to my liking. Thee are projects, e.g. MiniKube, that already give you lots of options. For a quick playground, they are fine. But for administrating a full cluster, I needed some advanced features. In addition, I wanted to install my own cluster, from scratch. Doing it myself is how I learn and understand all the ins and outs.

This process would be a trail-and-error journey, most of the way. So, I started with two virtual machines and went my path with dozens of snapshots. But soon, it became tedious. So, I wrote a small shell script in BASH that would allow me to automate some of my reset and restore procedures. Soon I had hammered down a couple of hundred lines of script code. Most of the code dealt with handling all the different options and validating input and output. Also, I did not put too much effort into a clean script. It should simply help me, but not replace everything covered by Vagrant or Libvirt.

Well, that last idea has vaporized. Because I saw myself confronted with an overly complex script for actually quite simple (sub)tasks. I started to extract some model data into a JSON file. But calling an external tool like `jq` and validating it wasn't any improvement, either.

## Writing a wrapper

So, shell scripts are fine. They get the job done. I made it work. Now, I wanted to make it right. I could have written a plain Kotlin or even Spring/Quarkus application. Both can optionally compile ahead natively, but still run in a JVM. Normally, spinning up a Spring or Quarkus application takes some time. Plain JVM applications need a JVM. Native applications are fast, but bundle everything, including the native runtime. Frankly, file size does not matter in this case.

But, I thought about this. Do I need all the boiler code from these major frameworks? Most of the time I make an external call to `virsh` or `vagrant`. I only needed some solid String manipulation and serialization. And, not to forget, some solid CLI handling. I wanted something small and simple. That's exactly where Kotlin Native comes to play.

## Kotlin Native

With Kotlin Native I have all the coding power of Kotlin at hand. Unfortunately compiling to a native executable makes a lot of Java libraries unavailable. But I do not need them, anyway. Kotlin Core is all I need. In addition, I added [kotlin.serialization](https://github.com/Kotlin/kotlinx.serialization) and [CLIKT](https://github.com/ajalt/clikt).

Now, I have a real binary, about 3MB big, with parameter completion to utilize my playground cluster. Everything regarding the state of the cluster or its configuration is stored in a JSON file which on every invocation is deserialized into `data` classes. Aside from the CLI handling, the code got quite simple and stripped down. And if something breaks, it either does not compile or I get a clear runtime error. Nice!

## Roadmap

For now, the project is still private. It is highly customized to my private network. I'll continue to integrate more settings and configuration options. When I think the project is ready for a public alpha release, I'll publish it. But I think this will be after I finished the [CKA](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/) exam ;-)
