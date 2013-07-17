---
layout: post
title: "Scala, SBT and IntellijIdea boilerplate"
date: 2013-01-16 23:33
comments: true
categories: productivity boilerplate
---
As I started delving into facets of Scala language I spent a lot of time in figuring out various tools for better productivity. Finally I was up and running with the basic setup though after lots of efforts. Therefore I am writing this post to help other newbies in setting up the basic project structure so that one may start being productive immediately. Before starting on how to setup things, please make sure you have <br />
1. [giter8]("https://github.com/n8han/giter8") installed. If you are using typesafe stack like me you will have it as typesafe-stack-folder/bin/g8. <br />
2. sbt 0.12.1 installed. 
<!-- more -->
Please note that sbt0.12 doesn't support scala 2.10 therefore this help is for those who wants to set up sbt 0.12 with scala 2.10. <br />
Alright let the fun begin now...<br />
+ run following command to download the template project <br />
``` g8 pkmishra/scala-sbt-idea-boilerplate.g8 ``` <br />
This will prompt you to give values to variables like project `name`, `organization`, `version` and `scalaVersion`. You may also select the default values. The template project is located at [scala-sbt-idea-boilerplate](https://github.com/pkmishra/scala-sbt-idea-boilerplate.g8) <br />
+ make directory called ~/.sbt/plugins and created file build.sbt if it doesn't exist already. <br/>
+ Write following lines to this build.sbt file so that sbt-idea plugin may be downloaded which is going to be used later... <br />

<p>
{% codeblock build.sbt lang:scala %}
import sbt._
import Defaults._

resolvers += "Sonatype snapshots" at "http://oss.sonatype.org/content/repositories/snapshots/"

libraryDependencies += sbtPluginExtra(
   m = "com.github.mpeltonen" % "sbt-idea" % "1.3.0-SNAPSHOT", // Plugin module name and version
   sbtV = "0.12",    // SBT version
   scalaV = "2.9.2"    // Scala version compiled the plugin
)
         
{% endcodeblock %}
</p>
+ Now go to the root of the project you have generated (e.g. if you have keep it default name should be basic-project) and run following command.<br />
sbt "gen-idea no-sbt-build-module" <br />
This will generate all the idea project files. <br />
+ Boom!! now you can open Idea IDE and open the project. To see the output just run command sbt run.
+ *Make sure to run following command everytime you add/remove a new dependency in your project build.sbt file.* <br />
sbt "gen-idea no-sbt-build-module" <br />
This will keep everything upto date and you won't get any compilation error in Intellij Idea. Hope this helps!!!