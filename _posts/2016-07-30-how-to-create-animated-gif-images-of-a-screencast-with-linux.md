---
layout: post
title: How to Create Animated GIF Images of a Screencast With Linux
tags: [linux, screencast, github]
date: 2016-07-30T01:40:55+02:00
---

Aside from just writing some simple description I also wanted to present a short screencast for my projects on github. After some rial an error I finally came up with the following solution:

- Create a screencast with SimpleScreenRecorder
- Convert the resulting video from SimpleScreenRecorder to a series of images
- Finally, convert and combine the series of images into one GIF file.

You can either compile [SimpleScreenRecorder](http://www.maartenbaert.be/simplescreenrecorder/) from [source](https://github.com/MaartenBaert/ssr) or install one of the provided [packages](http://www.maartenbaert.be/simplescreenrecorder/#download). In addition see the [wiki entry at ubuntuusers](https://wiki.ubuntuusers.de/SimpleScreenRecorder/).

    mplayer -ao null <videofile> -vo jpeg:outdir=/path/to/output

    cconvert /path/to/output/* -layers Optimize optimised.gif
    
Further size reductions can be achieved with:
    
    convert output.gif -fuzz 10% -layers Optimize optimised.gif
    
Check the `convert` man page for more information.

Regarding SimpleScreenRecorder you will have to play with the settings like frame rate, skipping frames and so on to find a setting that fits best for you.

