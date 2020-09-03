---
layout: post
title: Upgrading to Mysql 8.0
date: 2020-09-03T17:38:25+02:00
tags: [database, mysql, apt, linux]
---

For some days now, there is a new package in the debian repositories for Mysql. While the latest version installed was 5.7, the new one is a big jump to 8.0! Unfortunately a simple upgrade failed. It turns out one first has to run `mysql_upgrade` and after that do `apt upgrade`.

So, if you experienced some upgrade problems during an upgrade with APT, run the following first:

    #Check you can connection
    sudo mysql -h localhost -u <user> -p<passwd>
    
    #If you could login, exit and run
    sudo mysql_upgrade -h localhost -u <user> -p<passwd>
