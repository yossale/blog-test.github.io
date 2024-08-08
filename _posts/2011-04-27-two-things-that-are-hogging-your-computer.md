---
title: 'SearchIndexer &#038; TSVNCache : Two things that are hogging your hd - and your computer'
author: yossale

 
categories:
  - Off Topic
  - Programming
  - Tiddy Bits
---
I have a brand new Lenovo T510 , i7, 4G ,64bit , Windows 7 with an Nvidia GPU - and it sucks. In this price , with these specifications , it really shouldn't suck.  
I've tried numerous things (including the regular defrag scan disk sequence) , and I did find a few problems (It was utilizing only 3 GB out of 4 , for example) - But it still sometimes had the worst performances ever.

I then noticed the my Hard drive icon is constantly flashing , even when I do no intense IO operations. It's just always on. So I started searching for my HD hogger, and I found these two:

  * **SearchIndexer.exe** - This one is all MS fault. It's the MS file indexer on Windows 7 . Now ,  I really like the new"look for a command name instead of  searching the damn thing in tens of application" , but why does it have to hog my HD all the time? It was accessing my hd ALL THE TIME! .  
    [This is how you make it go away][1]. Basically , it's a service , so you just run the services.msc and stop it (and then kill it from the task manager).
  * **TSVNCache.exe** - This one only applies if you use the Tortoise SVN client. (If you have no idea what this is , you probably don't have it) . Apperantly , the Tortoise client really wants to be helpful , so it just keep scanning your entire hd to refresh the SVN icon on your files. This is such an over kill , as most of your folders aren't managed by the svn. [So this is how you make it stop crunching your hd , and only running where it should][2]

&nbsp;

My computer HD is now 90% lower on avg, and it actually feels like a good machine to have.

 [1]: http://www.howtogeek.com/howto/28450/what-is-searchindexer.exe-and-why-is-it-running/ "How Do You Stop This Process?"
 [2]: http://www.paraesthesia.com/archive/2007/09/26/optimize-tortoise-svn-cache-tsvncache.exe-disk-io.aspx "OPTIMIZE TORTOISE SVN CACHE (TSVNCACHE.EXE) DISK I/O"