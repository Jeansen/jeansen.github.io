---
layout: post
title: Independent CI with GitHub
date: 2022-01-30T09:57:37+01:00
tags: [ci, github, jenkins]
---

While preparing a new release for bcrm, I realized that my pipeline for Travis-CI no longer worked. That pipeline would run a small build script which would include a little chroot environment and wrap it together with the other release files. Unfortunately there is no free plan anymore. One can contact the Travis support team but that was already too much hassle for me. So, I checked Circle-CI. It is similar, but also different. Anyway, they have a free plan that one can directly use. And for my little setup, it would be more than enough.

Migrating from Travis-CI to Circle-CI is possible but also involves some time to read the docs and get the transition right. And who knows how long they will offer a free plan until they shift their philosophy as Travis did?

## Jenkins to the rescue

So, I said to myself. Wait a moment. I already have a CI setup which I utilize for testing on premise. Why not use it for release management, too? Almost everything is in place. And aside from Jenkins I only need two other things: 

    - A possibility to become aware of changes, so Jenkins can be notified
    - A way to upload/create a release

The first criteria is already resolved, too. I have Gitea installed on premise which regularly synchronizes itself with my projects on Github. When there is a new change (in this case a new tag), Gitea would notify Jenkins after synchronizing.

The only thing missing was a way to create releases and upload artifacts. There are plugins available for Jenkins, but I decided to utilize the REST API with plain old curl. As it turns out, only 2 calls are needed. One to create a release and another to add additional attachments.

## Reult

To wrap it all up: I created a simple Jenkins project that would only run when there are new Git tags. The shell script involved uses curl to create and upload releases. My local gitea mirrors my github project(s) and notifies Jenkins, whenever there are changes, so Jenkins can run the release job.

Works like a charm. No plugin, no alien CI/CD dependency. All under my control. And since Jenkins is open source supported by the Software in the Public Interest (SPI) non-profit organization, there is no vendor lock-in.
