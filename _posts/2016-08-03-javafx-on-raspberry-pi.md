---
layout: post
title: JavaFX on Raspberry PI
tags: [java fx, raspberry pi]
date: 2016-08-03T00:25:56+02:00
---

After I got my fingers dirty with some JavaFX programming, I also ever wanted to run JavaFX on my Raspberry PI. Especially to play with guestures. But things would be borring if it were just about installing Java and deploying the application.

## A verse of problems
So, why is it not just about installing Java and running a jar? At least, Oracle provides Java packagages for ARM plattforms. Isn't it all about write once, run everywhere?

Well, Oracle decided to no longer pack JavaFX with Java installation packages for ARM. Fortunately the folks at Gluon provide the missing libraries. But even then you have to know where to put them and how to start your application.

## A verse of challenges
I am greatful Gluon provides a compiled package but I do not know what their version reflects. As of this writing their versions is 8.60.7. If it reflects one of the tags in the OpenJFX repository (which would be 8u60-b07), then it has not been updated for some time now. To be preciese: Fri Mar 13 14:21:16 2015

The latest tag at the moment is 8u122-b00 created at Sun Jul 24 13:40:25 2016. So, I could have taken the easy way to just take the provided version by Gluon and continue playing with JavaFX on my little PI. But well, I do not really like dated software (for a lot of reasons) and also ever wanted to compile Java myself. Why not take the chance and start with JavaFX!

# One container to rule them all
After hours of reading accompanied by trial and error I finally came up with the following setup to build and deploy JavaFX as well as my JavaFX on my PI.

- Download the JavaFX repository with Mercurial
- run compilation with the help of a Docker container
- Deploy JavaFX application and JavaFX libraries with Maven
- Connect via SSH and run the application on the PI.

## Download JavaFX repository
In order to get the OpenJFX sources you will need mercurial. If you do not have it installed, please do so. For instance with `sudo apt-get install mercurial`. After that, clone[^1] the stable stream for JDK8 with:
    
    hg clone http://hg.openjdk.java.net/openjfx/8u-dev/rt
    
Before you can build the necessary libraries for ARM you need to prepare the download for cross compilation. The docker container will take care of that, but if you are interested in more background information see [Cross Building for ARM Hard Float](https://wiki.openjdk.java.net/display/OpenJFX/Cross+Building+for+ARM+Hard+Float).
  
[^1]: https://wiki.openjdk.java.net/display/OpenJFX/Building+OpenJFX#BuildingOpenJFX-GettingtheSources

## Build and run the Docker container
Here is a Docker container you can use for the building process. If you don't want to build it yourself, pull it from the docker registry.


    FROM debian
    
    RUN echo 'Acquire::http::proxy "http://local:3142";' > /etc/apt/apt.conf.d/02proxy
    
    COPY jdk8.deb /
    COPY gradle-1.8/ /gradle-1.8/
    
    RUN dpkg --add-architecture i386
    
    RUN apt-get update && apt-get install -y bison flex gperf libasound2-dev libgl1-mesa-dev \
        libgstreamer0.10-dev libgstreamer-plugins-base0.10-dev libjpeg-dev \
        libpng-dev libx11-dev libxml2-dev libxslt1-dev libxt-dev \
        libxxf86vm-dev pkg-config x11proto-core-dev \
        x11proto-xf86vidmode-dev libavcodec-dev mercurial libgtk2.0-dev \
        ksh libxtst-dev libudev-dev libavformat-dev qt5-default gdebi \
        ruby wget gcc-multilib g++-multilib zlib1g-dev:i386
    
    RUN gdebi jdk8.deb --option=APT::Get::force-yes=1,APT::Get::Assume-Yes=1 -n
    RUN rm /usr/lib/jvm/oracle-java8-jdk-amd64/jre/lib/ext/jfxrt.jar
    
    ENV JAVA_HOME /usr/lib/jvm/oracle-java8-jdk-amd64/
    ENV JDK_HOME /usr/lib/jvm/oracle-java8-jdk-amd64/
    ENV GRADLE_HOME /gradle-1.8/bin
    ENV PATH ${PATH}:$GRADLE_HOME

Run the docker container with a volume pointing the cloned repository of the JDK8 stable stream, e.g.:

    docker run
    
Now, this will take some time. Maybe you should get some fresh coffee ;-)

When everything is done, you should now have a folder ......


## Deploy JavaFX via SSH
With OpenJFX ready to be used we will now need to push it over to our friend "little PI". In my setup I did not have to do much after the initial setup with Raspian. That is, I downloaded the image, put it on a microSD and turned on the PI. Java 8 is already installed:

    pi@raspberrypi:~ $ java -version
    java version "1.8.0_65"
    Java(TM) SE Runtime Environment (build 1.8.0_65-b17)
    Java HotSpot(TM) Client VM (build 25.65-b01, mixed mode)

To make things simple, I created a demo JavaFX project and tuned a Maven POM to my needs. This is quite easy with the community editon of IntelliJ. Simply create a JavaFX project and convert it to a Maven project. To convert it to a Maven project right click on the top folder of your project in the project view and select "Add Framework Support ...". Select Maven, fill in the details, and you are done.

Next you should open the Maven view, right click on the maven entry for this project (there should only be one) and choose `Create 'settings.xml'`.

Here is my POM. In essence it uses ant to copy files via scp from my lokal machine over to little PI. That is, it will pack the demo application in a JARfile, create the necessary remote destination folders, copy over the files necessary for JavaFX and the JAR file containing the JavaFX demo application.

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
    
        <groupId>groupId</groupId>
        <artifactId>untitled2</artifactId>
        <packaging>jar</packaging>
        <version>1.0-SNAPSHOT</version>
        <name>simple javafx test</name>
    
    
        <build>
            <plugins>
                <plugin>
                    <!-- Build an executable JAR -->
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-jar-plugin</artifactId>
                    <configuration>
                        <archive>
                            <manifest>
                                <addClasspath>true</addClasspath>
                                <classpathPrefix>lib/</classpathPrefix>
                                <mainClass>sample.Main</mainClass>
                            </manifest>
                        </archive>
                    </configuration>
                </plugin>
                <plugin>
                    <artifactId>maven-antrun-plugin</artifactId>
                    <!--configuration>
                        <tasks>
                            <scp todir="${scp.user}:${scp.password}@${scp.host}:/${scp.dirCopyTo}" trust="true" failonerror="false">
                                <fileset dir="${bundle.dir}" />
                            </scp>
                        </tasks>
                    </configuration-->
                    <executions>
                        <execution>
                            <id>server-copy</id>
                            <goals>
                                <goal>run</goal>
                            </goals>
                            <phase>package</phase>
                            <configuration>
                                <target>
                                    <echo message="create remote folder for scp"/>
                                    <sshexec trust="yes" host="192.168.178.195" username="pi" password="raspberry" command="mkdir /tmp/armv6hf-sdk"/>
                                    <echo message="Push target jars to remote"/>
                                    <scp trust="yes"
                                         todir="pi:raspberry@192.168.178.195:/tmp/">
                                        <fileset dir="${basedir}/target">
                                            <include name="**/*.jar"/>
                                        </fileset>
                                    </scp>
                                    <echo message="jfx folder to remote"/>
                                    <scp trust="yes"
                                         todir="pi:raspberry@192.168.178.195:/tmp/armv6hf-sdk">
                                        <fileset dir="/home/marcel/tmp/docker_build/rt/build/armv6hf-sdk" />
                                    </scp>
                                </target>
                            </configuration>
                        </execution>
                    </executions>
                    <dependencies>
                        <dependency>
                            <groupId>org.apache.ant</groupId>
                            <artifactId>ant-jsch</artifactId>
                            <version>1.9.7</version>
                        </dependency>
                        <dependency>
                            <groupId>com.jcraft</groupId>
                            <artifactId>jsch</artifactId>
                            <version>0.1.53</version>
                        </dependency>
                    </dependencies>
                </plugin>
            </plugins>
        </build>
    </project>
    
## Run JavaFX on the PI
If everything goes well you can now establish a SSH connection to your PI and run the demo application with:

    sudo JAVAFX_DEBUG=1 java -Djava.ext.dirs=/tmp/armv6hf-sdk/rt/lib/ext/ -jar /tmp/untitled2-1.0-SNAPSHOT.jar 

Actually the `JAVAFX_DEBUG=1` property is not necessary when running the application from withing a ssh-session. But with this property you could also attach a keyboard (and mouse) to your PI and run the command above from within a terminal on your PI. If you want to stop the JavaFX application just hit `CTRL-C`.

And yes, you do need to run the command as root. Otherwise you will get some annying errors!