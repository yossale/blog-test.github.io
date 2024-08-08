---
title: Bug Update

date: 2009-01-07 11:33:00 +0800
categories: [Frameworks, Programming]
tags: [Hadoop, Hbase]
---
After a lengthy discussion with St.Ack (on the hbase channel in the mirc) and Jean Daniel about [this bug][1] , we currently believe that what we're seeing is " &#8230; a fumble of regionstate somehow.&#160; The master says its on regionserver X but when client goes there regionserver X says, I don't have it " (St.Ack). 

It seems the Master and the Regional Servers have some unresolved discrepancies which cause the client to access a region that doesn't exist. I'm waiting for the system to crush again to get a thread dump from the region server and the master , and maybe it will shed some light on matters.

In the mean while , stopping and restarting the client causes the Master and the RegionServers to resolve their issues - because on restart all regions are given out anew.

 [1]: https://issues.apache.org/jira/browse/HBASE-1094