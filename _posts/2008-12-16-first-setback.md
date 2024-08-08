---
title: First Setback

date: 2008-12-16 11:33:00 +0800
categories: [Frameworks, Programming]
tags: [Hadoop, Hbase]
---
We've tried loading the system with 10,000 , 100,000 and 200,000 files - everything worked perfectly.

We then moved on to running the full benchmark (11,00,000). After approx 400,000 files , HBase regionServer began to falter :

First of all , it seems that one regionServer (out of 2) was doing almost 90% of the work. Unsurprisingly , it is the one the faltered. (<span style="font-style: italic;">Nothing to do about it , apparently - as long as the META region is under a specific size , it is held on a single RS . After enough data , you'll have a split , and then the load will be more balanced</span>)

Then , I started to get this error msg :  


> org.apache.hadoop.hbase.NotServingRegionException</p>
The RegionServer is still running , but I'm not sure exactly what is wrong

<span style="font-weight: bold;">It began when the RS started a split :</span>

RegionServer log:  


> 2008-12-16 09:17:14,441 INFO org.apache.hadoop.hbase.regionserver.HRegion: Starting split of region</p>
HBaseMaster log:  


> 2008-12-16 09:17:16,902 INFO org.apache.hadoop.hbase.master.ServerManager: Received MSG\_REPORT\_SPLIT: obde_documententries,RQue7uxNoe59vJxljcd1rQ==,122937445  
> 9395: [B@674c5b37 from MYIP</p>
after a few split mechanisms issue all kind of info messeges , and everything seems OK , suddenly all the read/write request produce the above error.

<span style="color: rgb(0, 0, 0);">Resolution : After viewing several posts on the subject (specially </span><a style="color: rgb(0, 0, 0);" href="http://osdir.com/mljava.hadoop.hbase.user/2008-08/msg00042.html"><span style="text-decoration: underline; color: rgb(102, 102, 102);">this one</span></a><span style="color: rgb(0, 0, 0);">) , I tried to do the obvious thing and disable/enable the table. I'm not sure it resolved the problem or just delayed it. This solution solves the current problem , but it keeps happening again and again, I'm not sure why.</span>

<strike>  
<span style="font-weight: bold; font-style: italic; color: rgb(153, 153, 153);">Update: Probably my fault</span>

<span style="font-style: italic; color: rgb(153, 153, 153);">In my Java code (using the HBase API) I'm using the same instance of HTable created when I first started running. Apparently , the table object contains a map with the regions serving this specific table , and since I didn't update the HTable object , it didn't have the right regions.</span>  
<span style="font-style: italic; color: rgb(153, 153, 153);">I'll try that change and update</span></strike>