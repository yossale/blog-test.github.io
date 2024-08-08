---
title: Still unresolved

date: 2008-12-25 11:33:00 +0800
categories: [Frameworks, Programming]
tags: [Hadoop, Hbase]
---
We've been working on the previous bug (last post) for a few days now , but unfortunately nothing is working. We've suspected the error might be caused by wrong insertion sequence on our side , but it seems very unlikely now (we checked everything is corresponding with the API , reviewed it with others and all was fine. Besides , it's really not that big a code)

Today (Actually , last night) we recieved a similiar error (but not the same) -  


> org.apache.hadoop.hbase.NotServingRegionException: Region test2,a521DfAPKkUbWqIOHc8pAQ==,1230151003797 closed  
> at org.apache.hadoop.hbase.regionserver.HRegion.obtainRowLock(HRegion.java:1810)</p>
My guess is that once again it has something to do with some race condition that causes some split not to finish correctly , and thus attempts to perform various actions on the should-have-been-healthy Region (getRegionName() , or in this case obtainRowLock() ) to fail, and all other attempts to access the same region return the same result. It appears to be some kind of a related bug to [this bug.][1]

Since it only happens after a lot of insertions in a very rapid rate (about 500 small files per minute , usually fails after a couple of millions) , we will have to consider if it's good enough for us - since the real life conditions should be a much less harsh.

 [1]: https://issues.apache.org/jira/browse/HBASE-1061?page=com.atlassian.jira.plugin.system.issuetabpanels:comment-tabpanel&focusedCommentId=12656385#action_12656385