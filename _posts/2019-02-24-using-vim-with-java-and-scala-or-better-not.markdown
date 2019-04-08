---
layout: post
title: Using Vim with Java (and Scala) - or better not?
date: 2019-02-24T16:05:00+01:00
tags: [vim, scala, java]
---

Every then and when I try to see if it might be possible to get some support for languages in Vim that I would normally us a professional IDE for.

For all my scripting Vim is the tool of choice. But for Java and Scala, I use IntelliJ. Regarding Vim I'd be happy to just have intelligent code completion and linting.

Linting I did not try. I'll postpone that to another day. But code completion I tried with `ensime` and `eclim`. Sadly, none really worked (if at all). And after a couple of hours trying, I gave up.

## Eclim

Eclim actually was quite easy to install. I download the right Eclipse version and the Eclim binary. After extracting Eclipse I ran Eclim and followed the instructions. Then I only had to get the vim plugin working with my plugin manager. But that one I solved quite swiftly.

Then I booted the Eclim server and opened one of my Scala projects. I had to call `:ProjectCreate <path> -n scala` to have Eclim give me code completion when using the `Insert mode completion`. So, doing `Ctrl-X Ctrl-U` actually worked. Once! Then the server got an exception.

Also, you are stuck with Eclipse 4.8. Any newer version did not work. And I do not know how it is with different Java versions.

## Ensime

Ensime is a project that mainly targets Emacs but also offers a lot of functionality for Vim and others. This one is even easier to setup. Following the instructions on their page I installed some Python packages via `pip` and the Vim plugin. The only thing missing was calling `:EnInstall` and tweaking Vim for code completion. 

And that's where the trouble started. Actually, the Ensime project does not say anything about code completion. Where Eclim offers code completion without any special configuration (and also offers help for integration with some prominent Vim plugins), there is no such documentation for Ensime. Anyway it was not hard to find some hints for `Deoplete`. But for me, nothing I found and tried actually worked.

Even more frustrating was the fact that the procedures offered via the SBT plugin for generating new configurations seems to be broken, too. I've cleared all my caches and removed all configuration files. Then I ran SBT and created new configurations with `ensimeConfig` and `ensimeConfigProject`. But when I then called `:EnInstall` from within Vim, it complained about missing (cached) jar files.

So, for now I'll put this challenge aside and try another time, again.