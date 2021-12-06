---
layout: post
title: Working with scaladoc and IntelliJ
date: 2019-02-18T15:35:10+01:00
tags: [scala, intellij]
---

While maven works out of the box you might have to tweak scaladoc when using SBT. Here's what you have to do.

## SBT and scaladoc

Working on my little scalafx project (which I started with Java 8 and plain JavaFX), I faced the problem that scaladoc hints in IntelliJ did not work.

The non-scala project had something like this in its POM file:

    <dependencies>
    
        <!-- https://mvnrepository.com/artifact/org.openjfx/javafx-base -->
        <dependency>
          <groupId>org.openjfx</groupId>
          <artifactId>javafx-base</artifactId>
          <version>11</version>
        </dependency>
        <dependency>
          <groupId>org.openjfx</groupId>
          <artifactId>javafx-controls</artifactId>
          <version>11</version>
        </dependency>
        <dependency>
          <groupId>org.openjfx</groupId>
          <artifactId>javafx-fxml</artifactId>
          <version>11</version>
        </dependency>
        <dependency>
          <groupId>org.openjfx</groupId>
          <artifactId>javafx-graphics</artifactId>
          <version>11</version>
        </dependency>
        <dependency>
          <groupId>org.openjfx</groupId>
          <artifactId>javafx-media</artifactId>
          <version>11</version>
        </dependency>
        <dependency>
          <groupId>org.openjfx</groupId>
          <artifactId>javafx-swing</artifactId>
          <version>11</version>
        </dependency>
      </dependencies>
      
And that worked totally fine.

In scala I had to do something like this:

    // Add dependency on ScalaFX library
    libraryDependencies += "org.scalafx" %% "scalafx" % "11-R16"

    // Determine OS version of JavaFX binaries
    lazy val osName = System.getProperty("os.name") match {
      case n if n.startsWith("Linux")   => "linux"
      case n if n.startsWith("Mac")     => "mac"
      case n if n.startsWith("Windows") => "win"
      case _ => throw new Exception("Unknown platform!")
    }
    
    lazy val javaFXModules = Seq("base", "controls", "fxml", "graphics", "media", "swing", "web")
    libraryDependencies ++= javaFXModules.map( m =>
      "org.openjfx" % s"javafx-$m" % "11" classifier osName withSources() withJavadoc()
    )
    
Almost everything is as described on the [project page](https://github.com/scalafx/scalafx#scalafx-11) with the exception of `withSources() withJavadoc()`. Whatever I tried in the SBT shell, it did not help. Only after adding those two function calls I also had the scaladoc available.

Unfortunately IntelliJ will not add the sources to your project automatically. You will have to add them yourself. Navigate to `File -> Project Structure` and select `Libraries`. There you will find a list of installed libraries by SBT, for instance: `sbt: org.openjfx:javafx-controls:11:linux:jar`. If you select an entry from that list you can add the documentation on the right side. 

For the `javafx-controls` package the `Classes` path on my system is:

    ~/.ivy2/cache/org.openjfx/javafx-controls/jars/javafx-controls-11-linux.jar
    
The `JavaDoc` path now is:

    ~/.ivy2/cache/org.openjfx/javafx-controls/docs/javafx-controls-11-javadoc.jar
    
Et Voilà! Using `Ctrl-Q` on any function now gives me the attached documentation.
