---
title: Begin Experimenting

date: 2008-12-14 11:33:00 +0800
categories: [Frameworks, Programming]
tags: [Hadoop, Hbase]
---
Today we started the production-mode experiemnts.

In phase 1 we will have only 4 servers:

A: 80 GB , 8 GB Ram - will contain the Hadoop NN and Hbase Master  
B: 45 GB , 4GB Ram - will contain the SNN and a DN  
C,D : 45 GB , 4GB Ram - will contain (each) a DN and a RS (RegionalServer)

In a few days we will add 3 more servers like C & D (E , F ,G) which will contain only DN.

Currently , both Hbase and Hadoop are up and running.

We will now start to load the HBase with ~10,000,000 files (~30 GB) and we'll test how it handles it.