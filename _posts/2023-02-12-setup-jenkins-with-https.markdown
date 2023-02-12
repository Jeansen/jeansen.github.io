---
layout: post
title: Set up Jenkins with HTTPS
date: 2023-02-12T23:08:22+01:00
tags: [jenkins, https, linux]
---

Here is a quick guid on how to set up Jenkins with HTTPS. There are numerous guides available, already. And I read some of them. But two things struck me odd. First, some guides want you to change the central systemd file for Jenkins (instead of an override file) and secondly, they want you to create a JKS file (instead of PKCS 12).

So, my words of warning: Never overwrite files in `/etc/systemd/system`. These files are manged by your system and package manager. If you want to add or change settings for Jenkins, use [/etc/systemd/system/jenkins.service.d/override.conf](https://www.jenkins.io/doc/book/system-administration/systemd-services/). JKS is a proprietary format. [PCKS #12](https://en.wikipedia.org/wiki/PKCS_12) on the other hand, is an [industry standard](https://en.wikipedia.org/wiki/PKCS) and also the default keystore format since Java 9!

### Generate SSL certificate

First, you'll need a certificate. You can create one with openssl:

     openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out jenkinns.pem

Naturally, if you already have a certificate, you do not need to create a new one. But you'll have to convert it to a PKCS #12 file.

Continuing with the example above, you'll have to merge the files `key.pem` and `jenkins.pem` in a PKCS #12 keystore:

    openssl pkcs12 -inkey key.pem -in jenkins.pem -export -out jenkins.p12

Finally, put the file `jenkins.p12` somewhere accessible to Jenkins. I put it in `/var/lib/jenkins`.

    sudo mv ./jenkins.p12 /var/lib/jenkins
    # I assume you run Jenkins with default settings. If you run it with a different user, you'll have to adapt the 'chown', of course!
    sudo chown jenkins:jenkins /var/lib/jenkins/jenkins.p12

### Enable HTTPS in Jenkins service

With a new and shiny keystore in place, tell Jenkins how to use it. Create the file `/etc/systemd/system/jenkins.service.d/override.conf` and put the following content in it.
    
    [Service]
    Environment="JENKINS_HTTPS_PORT=8443"
    Environment="JENKINS_HTTPS_KEYSTORE=/var/lib/jenkins/jenkins.p12"
    Environment="JENKINS_HTTPS_KEYSTORE_PASSWORD=<your-password>"

Now, restart the Jenkins service.

    sudo systemctl daemon-reload
    sudo systemctl restart jenkins.service 

### All done

Accessing `https://localhost:8443` now should give you the Jenkins login page, but with a valid and secure https connection.
