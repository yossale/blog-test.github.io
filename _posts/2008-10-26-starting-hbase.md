---
title: "Testing Hadoop - Starting HBase"

date: 2008-10-26 11:33:00 +0800
categories: [Frameworks, Programming]
tags: [Hadoop, Hbase]
---

Hbase configuration and running is very similar to hadoops. Not surprisingly , they also have [a nice Getting Started page][1]. The tricky part , though , is to understand what the hell do they mean with all these"column families" , and why the syntax is not plain SQL.

The bottom line is this : HBase is NOT a relational DB. It's more like a hash of hashes (more specifically , it's a sorted map). It took me quite a while to figure out what the hell they want , and it was mainly due to [this great article][2] (thank you Jim!)

I've started playing with the HBase shell , which proved very good hands-on practice for later. After tweaking a little with the shell , I've tried to use the Java API (The getting started page has a couple of examples on how). It's fairly simple , but there are (apparently) a few ways to input data to HBase , and I haven't found a complete survey of these methods and their ups and downs yet.

 [1]: http://hadoop.apache.org/hbase/docs/current/api/overview-summary.html#overview_description
 [2]: http://jimbojw.com/wiki/index.php?title=Understanding_Hbase_and_BigTable
