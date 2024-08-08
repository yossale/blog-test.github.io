---
title: Can’t live with it

date: 2008-12-31 11:33:00 +0800
categories: [Frameworks, Programming]
tags: [Hadoop, Hbase]
---
We assumed that the bug was caused mainly because of the high load rate , and that once the bulk of the data will be in HBase and the load will drop considerably , we won't see it again.

Unfortunately , That didn't happened. The bug happened again on our production-like environment , and that's not something we could live with. As I mentioned before , similar issues seem to already be on the HBase Jira ([HBASE-1061][1], [HBASE-1094][2] - opened by me , [this thread][3]) .

To the best of my understanding , what actually happens is this: The batchUpdate is trying to address a region that has already been closed (Probably by a split) . We're quite sure the problem is on the client-side - because restarting the client will usually solve the problem. 

  * First option : the batchUpdate request is being done just a moment before the split is initiated , an thus it gets a reference to a region that was running when the request was performed , but it was closed soon afterward. I don't really think this is the case because to the best of my log files , sometimes the failure happends not during any split.</p> 
  * Second option : the batchUpdate is using a stale regionServers map - which brings up the question"why?" , when was it supposed to update , and why didn't it do it? 
  * Third option : the .META. table holds stale regions data - i.e there was a split , but it wasn't updated. I find it hard to believe this is the case , becuase restarting the client solves the bug , and this behaviour can not be explained in this case.

We decided to try and solve the bug (or try until someone else solves it) - I'll try to keep updating about our progress

 [1]: https://issues.apache.org/jira/browse/HBASE-1061
 [2]: https://issues.apache.org/jira/browse/HBASE-1094
 [3]: http://markmail.org/search/?q=BatchUpdate+hbase+Region#query:BatchUpdate%20hbase%20Region+page:1+mid:m7a4nms5fef422ry+state:results