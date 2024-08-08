---
title: "Testing Hadoop - Starting Hadoop"

date: 2008-10-26 11:33:00 +0800
categories: [Frameworks, Programming]
tags: [Hadoop, Hbase]
---

First thing : It works. 
If you're getting a lot error messages and you start thinking "well , maybe it's crap. still 0.18 can't be that good a version" Stop. It works.

Hadoop has a [very good Getting Started page][1] , which will probably tell you everything you need to know. 
It requires a little tweaking with configuration files , but nothing big , really.

Do notice one thing (which I found out the hard way): Do NOT use "localhost" in the configuration files for anything that requires listening 
to ports. Meaning, **"localhost" != "the ip of this machine"**. This minor point caused me a lot of grievances , server A was suppose to listen on port 6200 , 
and server B was (supposed to) communicate with it through that port. However, since I specified "localhost:6200" at server A conf file , 
they couldn't communicate.

Another annoying ... amm ... thingy : when you format the namenode (hadoop namenode -format) you get a "Are you sure" (Y/N)? message. 
Press "y" & the format fails and aborted.  
Why? because it should have been "Y". This is not even a bug , but boy is that annoying - especially when you're a beginner , and has no idea what the hell is going on.

Last point : Hadoop & HBase version number are messed up. Really. I started with Hadoop 0.18 and HBase 0.18 (Seems quite reasonable, ahh?) - And after losing a few more hairs, I discovered this page. Do yourself a favor and consult this page BEFORE you start installing.

 [1]: http://wiki.apache.org/hadoop/GettingStartedWithHadoop
