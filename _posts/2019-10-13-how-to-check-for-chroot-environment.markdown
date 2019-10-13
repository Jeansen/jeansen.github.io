---
layout: post
title: How to check for chroot environment
date: 2019-10-13T01:36:30+02:00
tags: [debian, bash, shell, linux]
---

This will be short but hopefully of interest. Today I had the need to find out if my script is being executed in a chroot environment.

I found two interesting solutions. One is very portable, the other is - as far as I can tell - only available on Debian-based systems.

# The Debian way

To find out if your script runs from withing a chroot environment you can use `ischroot`. Here's an excerpt from the man page regarding the different return codes:

       0      if currently running in a chroot
       1      if currently not running in a chroot
       2      if the detection is not possible (On GNU/Linux this happens if the script is not run as root).
       
A simple one-liner might look like this:

    ischroot && echo "running in chroot"

# The portable way

Another solution I found simply checks for the inode of `/`. This should always be 2. But in a chroot environment it will be much bigger.

To check for the inode number you can us `ls -id /`. Following is a rewrite of the previous one-liner:

    [[ $(ls -id / | awk '{print $1}' -gt 2 ]] && echo "running in chroot"
    
## When the portable way does not work

If your chroot environment is on its own partition, the heuristic presented above will not work. The same is true for systems that us random inode numbers, for instance OpenBSD.
        

