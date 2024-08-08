---
title: We’ll try to live with it

date: 2008-12-28 11:33:00 +0800
categories: [Frameworks, Programming]
tags: [Hadoop, Hbase]
---
As you remember , HBase tends to collapse (Return"NotServingRegionException") after a few millions of files (Latest crash: 6 million).  
Since we (want to) believe this only happens because of the rapid insertion rate (~500 a minute) , we will try to load all the files into the the HBase , and then test it in Production-Like mode: meaning , mainly read request , and much lower insertion rate.

However , I must say that this bug makes me very uncomfortable with using HBase in production environment. If the data was real-time critical I probably would have tried to find some workaround that would use Hadoop only - at least until the bug is fixed.

We'll start the Production-Like environment soon , and I'll update about the result.