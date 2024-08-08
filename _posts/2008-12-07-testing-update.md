---
title: Testing update

date: 2008-12-07 11:33:00 +0800
categories: [Frameworks, Programming]
tags: [Hadoop, Hbase]
---
We are planning to move on to production soon , and we intend to build a test environment that will resemble the production environment as close as possible.

We will begin with 5 dedicated Linux machine , that will hold the NN,SNN,DN , HBase master and HBase RegionServers. Once we build the configuration , we will start benchmarking and testing for failover and failback capabilities.