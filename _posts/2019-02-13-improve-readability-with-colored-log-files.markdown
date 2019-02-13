---
layout: post
title: Improve readability with colored log files
date: 2019-02-13T10:37:27+01:00
tags: [linux, bash]
---

For some days now I had to read a lot of log files. For better visual filtering I did some research and found some promising tools. But after some fiddling around, none of them could fit my needs. Either they were buggy in some way, too limited or added too much overload. And in the end I needed something portable, anyway.

## Possible solutions
For instance, `lnav` is a very interesting tool. But I had a lot of problems with larger files. It happend too often that the application just crashed or "froze to death".

Another tool I tried was `grc`. That one is really neat. Especially when it comes to live reformatting (which was [broken for me in some parts](https://github.com/garabik/grc/issues/104)). But even without that it failed with my setup. While it works great with `tail -F`, it [did not work as expected with `less`](https://github.com/garabik/grc/issues/103). Either it was terribly slow or it failed when following a file. But, for static views or with `tail` it is a tool I can only recommend. It even has some nice wrappers you can use to colorize a lot of well-known command outputs like `ls` or even `docker`.

And, as I would be using these tools not only locally but on different servers (where I have limited rights), I actually needed a tool that I could run without having to install anything.

## The one and only solution

So, here is the inception of my solution that simply uses `sed` and `bash`.

    #!/usr/bin/env bash
    
    trap "exit" INT TERM ERR
    trap 'kill $C_PID' EXIT
    
    ramfile="$(mktemp -p /dev/shm/)"
    
    tail -F -n +1 "$1" | sed -u 's#^[0-9]\+\(-[0-9]*\)*\s[0-9]*\(:[0-9]*\)*#\x1b[30;48;5;15m&\x1b[0m#; s#\bWARN#\x1b[30;48;5;226m&\x1b[0m#; s#\bERROR#\x1b[15;48;5;1m&\x1b[0m#; s#INFO#\x1b[96m&\x1b[0m#;' > "$ramfile" &
    C_PID=$!
    
    
    sleep 0.1 #otherwise less might not show anything
    less -R -N "$ramfile"
  

Of course, this is only a first draft, but it already adheres to the following criteria:

  - It is portable because only linux core tools are needed. Actually, it could be even more portable by using `sh` instead of `bash`.
  - It is very efficient because with the utilization of `/dev/shm` temporary files are stored in virtual memory instead only.
  - It allows full customization of color and formatting (if need be)
  - It works with `less`.
  
With this little script at hand I was then able to run it like this `./lesser.sh some-logfile.log` which would show me a custom colored log file in `less` and even allow me to follow the file (Shift-F).

## Where to go from here

This script is a first start. There is still a lot to be done to make it robust. But with time I believe it could prove to be a valid addition to the other tools I tried and partly use. Another project to be started on github ... 
  
   




