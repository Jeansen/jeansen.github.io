---
layout: post
title: Loading resources in Kotlin
date: 2021-10-31T21:53:38+01:00
tags: [Kotlin]
---

I've just finished converting a little Scala project of mine to Kotlin. Though the transition worked quite well, there was one thing that took me some time to
figure out.

Both projects follow a general layout with respect to where I put my resource files. In both projects the path is `/src/main/resources`. Source code files
reside in `/src/main/scala` and `/src/main/kotlin` respectively.

In the Scala code, I had a Class with the following call inside a method, which worked quite well:

    getClass.getResource("chrome.css").toExternalForm

To have this work in Kotlin, I had to take multiple aspects into account:
    - I had to prepend a '/'
    - I ahd to provide the full path below the 'sources' folder.
    - I needed different calls depending on weather I loaded the resource from within a lambda or property/method.

The full path to `chrome.css` is `/src/main/resources/local.net.ui/chrome.css`. That is, under the `resources` folder, there is another folder
named `local.net.ui` and within that lies `chome.css`.

Inside a Class (in my case, it is named `Chrome`), I could load the `chrom.css` file with a property initialization or directly from within the lambda where it
is needed:

    #From within a lambda
    val css: String? = Chrome::class.java.getResource("/net.ui/chrome.css")?.toExternalForm()

    #As a class property (or from witin a method)
    val css: String? = javaClass.getResource("/net.ui/chrome.css")?.toExternalForm()


Happy coding!
