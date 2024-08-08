---
title: "Testing Hadoop - Problem Definition"

date: 2008-10-26 11:33:00 +0800
categories: [Frameworks, Programming]
tags: [Hadoop, Hbase]
---


We have a very large amount of relatively small files (~5k avg , 41k max , 0k min) , that we access a lot (20M times a day) for various computations. Currently , all the data is stored on a single server - a very ad-hoc solution that was OK until now , but is no longer acceptable - in terms of Service level , redundancy , backup , and so forth. We add approximately 5K new files a day , and data is never deleted .

**We want a system that supply:**
1. High availability  
2. Easily scalable  
3. Inbuilt support for /mechanisms of - Backup & Replication  
4. Load balancing  
5. Fail-over mechanism  
6. A minimal Downtime is acceptable (90% SLA , not 99%. We can make-do without the system for short , non-frequent periods of time )

We reviewed several options , and **the 3 finalist were**
1. [EHCacheServer][1]  
2. Using [TerraCotta ][2]and [EHCache ][3]( + an in-house layer of hash to map files to specific machines)  
3. [Hadoop ][4]+ [HBase][5]

EHCacheServer sounds like the perfect solution , but we found so very little information about it that it spooked us a little. We know, use and like the EHCache solution , but the lack of samples and more documentation on the server is alienating.

Using TerraCotta and EHCache ( + an in-house layer of hash to map files to specific machines) : some of us still think this might be a better solution , but the general feeling is that forcing the TC solutions to what we want is too far from the TC main idea , and it bound to effect offer efforts.

Hadoop + HBase seems like it was literally built for us : it has High availability , it's easily scalable , the Hadoop replication mechanism is a great Backup & replication solution and the replication also enable easy load balancing .  
What did trouble us is the lack of Fail-over mechanism. However , we decided that since HAdoop & Hbase do have partial mechanisems to support fail-over (The secondarynamenode , FS Image , and so forth) , we think we can build our on Fail-over mechanism using Linux Heartbeat.

So we set off to start the Hadoop & HBase solution benchmark. This is it's story <img src="http://ams18.siteground.eu/~yossale4/wp-includes/images/smilies/simple-smile.png" alt=":)" class="wp-smiley" style="height: 1em; max-height: 1em;" />

 [1]: http://ehcache.sourceforge.net/documentation/cache_server.html
 [2]: http://www.terracotta.org/
 [3]: http://ehcache.sourceforge.net/index.html
 [4]: http://hadoop.apache.org/core/
 [5]: http://hadoop.apache.org/hbase/
