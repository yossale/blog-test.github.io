---
title: Hadoop Failover and Failback Solution

date: 2008-12-03 11:33:00 +0800
categories: [Frameworks, Programming]
tags: [Hadoop, Hbase]
---
As you can see in the previous blog , Hadoop successfully failover , but it fails to failback.  
Our solution to this problem is this:

The failover will be done as described in the [post a couple of days ago][1].  
The failback will have to be done manually (it makes sense , as we will probably need to figure out why the server crushed in the first place)

I will update about the specifics as soon as we have it settled , and I'll do my best to document the testing and integration process.

 [1]: http://myhadoopnhbaseexperience.blogspot.com/2008/11/hadoop-failover-and-hopefully-failback.html