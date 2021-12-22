---
layout: post
title: From Xorg to Wayland
date: 2021-12-17T23:31:49+01:00
tags: [linux, gnome, xorg, wayland]
---

A quick journey history of my transition from Xorg to Wayland. I waited with this transition as long as possible because two of my favorite tools do not work on 
Wayland. But now it seems that my favorite dock [Plank](https://github.com/ricotz/plank) is broken. And actually, [Xorg server is a mess. That's, why Wayland was
introduces.](https://wayland.freedesktop.org/faq)

So, I had to find two replacements. A natural replacement for plank was [Dash-To-Dock](https://extensions.gnome.org/extension/307/dash-to-dock/). It does not 
have the beloved fish-eye effect, but the rest is very similar. 

The other tool I use very frequently is an implementation of pie menus. Luckily, there is a Wayland project of the same author called [Fly-Pie](https://extensions.gnome.org/extension/3433/fly-pie/).
But it is only available for GNOME. The native X11 application [Gnome-Pie](https://github.com/Schneegans/Gnome-Pie) primarily targets GNOME, but not exclusively.

Time to switch!

## System failure

Right! It should be a matter of seconds. Guess what? Murphy's Law punched me right in the face. First problem was, that GNOME Shell
could not connect to X server. Problem was, I still had the legacy wrapper for Xorg installed (xserver-org-legacy). Uninstalling helped to get a connection. And 
with Wayland I do want nor need any X server for my user session.

## Broken extension

Still, after punching in my password GDM simply did nothing. I couldn't even use my pointer.
Turns out, one of my extensions was incompatible with Wayland. After I turned all extensions off, I could successfully log in. Then I activated extension 
by extension, logged out and back in until I found the culprit. Actually it is a sad thing that things like reloading the Gnome Shell [no longer work](https://mail.gnome.org/archives/commits-list/2015-March/msg01019.html).

## Global shortcuts

The reason the author of Gnome-Pie rewrote his tool from scratch was driven by the fact that it could not simply be ported to Wayland. Xorg's security concept is
completely different to Wayland's. The latter one is more secure and more strict. The very reason why it was crated, in the first place.

This also causes some tools to no longer work the same way as they did with Xorg. For instance, I use [guake](https://github.com/Guake/guake) for fly-in terminals 
on an every-day basis. In quake, you can simply define a shortcut to bring up a terminal. That works like a charm within Xorg, but not with Wayland. As soon, as 
a window looses focus (disappears), you can no longer bring it back with a predefined shortcut.

The solution was to delete the shortcut set in guake's preferences and create a keyboard shortcut in Gnome that triggers guake. Effect is the same, but it took me 
some time to figure that one out or better say find it somewhere in the [comments](https://github.com/Guake/guake/issues/1642).

The same is true for [autokey](https://github.com/autokey/autokey). Global shortcuts to run a command won't work. You'll have to define them via custom shortcuts 
managed by your desktop environment, like GNOME. But application shortcuts still work as expected, as far as I can tell.
