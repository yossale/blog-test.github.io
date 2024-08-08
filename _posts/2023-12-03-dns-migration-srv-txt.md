---
title: 'MongoDB Cluster Switching: Leveraging TXT and SRV Records'
description: How to do a DNS migration between MongoDB clusters using SRV and TXT records.
date: '2023-12-03 10:00:00'
image:
  path: /assets/images/2023-12-03-dns-migration-srv-txt/srv-txt-cover.png
  alt: 
---

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> The views expressed in this post are my own and do not reflect the official policy or position of MongoDB, Inc. The information provided is based on my personal experience and is intended as general guidance. These guidelines are not a substitute for professional advice and should not be relied upon as such.
{: .prompt-tip }

<!-- markdownlint-restore -->

## Problem:

We have a web app called App, with several microservices, all served from a MongoDB cluster called `OLD-CLUSTER`. 

We need to migrate to a new MongoDB cluster called `NEW-CLUSTER`, but we can not take down all the microservices at once to update their db connection string (too much downtime).

![Problem](/assets/images/2023-12-03-dns-migration-srv-txt/srv-how.png)

We want to find a way to do a DNS migration and just redirect the current URI to point to the new cluster.

## TL;DR

It's possible, but a "regular" host redirection won't work. Because MongoDB uses SRV+TXT records to direct the drivers to the correct host, you'll need to create an SRV record with the hostnames of some of the replica-set members, a TXT record with the replicaSet id, and restart your applications. 

You can use the official explanation [here](https://www.mongodb.com/developer/products/mongodb/srv-connection-strings/).

Keep in mind if the replica set's members' hostnames will change, you'll have to keep the SRV record updated. 

## In Depth

Starting from 3.6, the MongoDB driver uses the SRV protocol to create the initial connection with the MongoDB cluster. When the driver connects to your cluster `oldcluster.abcde.mongodb.net`, it starts by asking 
 
> Hey, can I get the SRV record with all the hosts relevant to `oldcluster.abcde.mongodb.net`?

```
nslookup -type=SRV _mongodb._tcp.oldcluster.abcde.mongodb.net

Server:         127.0.0.1
Address:        127.0.0.1#53

Non-authoritative answer:
_mongodb._tcp.oldcluster.abcde.mongodb.net      service = 0 0 27017 oldcluster-shard-00-03.abcde.mongodb.net.
_mongodb._tcp.oldcluster.abcde.mongodb.net      service = 0 0 27017 oldcluster-shard-00-02.abcde.mongodb.net.
_mongodb._tcp.oldcluster.abcde.mongodb.net      service = 0 0 27017 oldcluster-shard-00-00.abcde.mongodb.net.
_mongodb._tcp.oldcluster.abcde.mongodb.net      service = 0 0 27017 oldcluster-shard-00-01.abcde.mongodb.net.
```

> Can I get the TXT record with the replicaSet ID I'm supposed to connect to?

```
nslookup -type=TXT oldcluster.abcde.mongodb.net                                         

Server:         127.0.0.1
Address:        127.0.0.1#53

Non-authoritative answer:
oldcluster.abcde.mongodb.net    text = "authSource=admin&replicaSet=atlas-3cr2aq-shard-0"
```

And now the driver knows which hosts it needs to connect to - e.g: `oldcluster-shard-00-02.abcde.mongodb.net`, and which replicaSet (and additional properties) it needs to apply - `authSource=admin&replicaSet=atlas-3cr2aq-shard-0`. 

So, if you want to use a different hostname, what you need to change are the SRV + TXT entries of the URI, and restart your app. Let's see how.

## Hands-on

We have two clusters: `OLD-CLUSTER`, and `NEW-CLUSTER`. We want to update our DNS such that when a MongoDB driver reaches for `oldcluster.abcde.mongodb.net`, they'll connect to NEW-CLUSTER. 

### Step 1 - Get the NEW-CLUSTER SRV records

Run `nslookup` on the address of NEW CLUSTER - something like this:
```
> nslookup -type=SRV _mongodb._tcp.newcluster.abcde.mongodb.net

Server:         127.0.0.1
Address:        127.0.0.1#53

Non-authoritative answer:
_mongodb._tcp.newcluster.abcde.mongodb.net      service = 0 0 27017 newcluster-shard-00-03.abcde.mongodb.net.
_mongodb._tcp.newcluster.abcde.mongodb.net      service = 0 0 27017 newcluster-shard-00-02.abcde.mongodb.net.
_mongodb._tcp.newcluster.abcde.mongodb.net      service = 0 0 27017 newcluster-shard-00-00.abcde.mongodb.net.
_mongodb._tcp.newcluster.abcde.mongodb.net      service = 0 0 27017 newcluster-shard-00-01.abcde.mongodb.net.
```

In this case, the SRV record looks like this:
```
0 0 27017 newcluster-shard-00-03.abcde.mongodb.net.
0 0 27017 newcluster-shard-00-02.abcde.mongodb.net.
0 0 27017 newcluster-shard-00-01.abcde.mongodb.net.
0 0 27017 newcluster-shard-00-00.abcde.mongodb.net.
```
The structure is `priority, weight, port, target` - so in our case, all hosts have the same priority and weight (0), and they all listen on 27017.

### Step 2 - Get the NEW-CLUSTER TXT record

To get the TXT record, we'll do something similar:

```
> nslookup -type=TXT newcluster.abcde.mongodb.net                                         

Server:         127.0.0.1
Address:        127.0.0.1#53

Non-authoritative answer:
newcluster.abcde.mongodb.net    text = "authSource=admin&replicaSet=atlas-6tt1bq-shard-0"
```

So our text record is just `authSource=admin&replicaSet=atlas-6tt1bq-shard-0`

### Step 3 - Update the DNS service

Now we can go to our DNS service (in my case, Route53), and create the following records:

![Route53](/assets/images/2023-12-03-dns-migration-srv-txt/srv-txt-route53.png)

Wait for a couple of minutes for the DNS records to propagate, and you're done - when reaching out for the `OLD-CLUSTER`, you'll get the information of `NEW-CLUSTER`. 

```
> nslookup -type=SRV _mongodb._tcp.oldcluster.abcde.mongodb.net

Server:         127.0.0.1
Address:        127.0.0.1#53

Non-authoritative answer:
_mongodb._tcp.newcluster.abcde.mongodb.net      service = 0 0 27017 newcluster-shard-00-03.abcde.mongodb.net.
_mongodb._tcp.newcluster.abcde.mongodb.net      service = 0 0 27017 newcluster-shard-00-02.abcde.mongodb.net.
_mongodb._tcp.newcluster.abcde.mongodb.net      service = 0 0 27017 newcluster-shard-00-00.abcde.mongodb.net.
_mongodb._tcp.newcluster.abcde.mongodb.net      service = 0 0 27017 newcluster-shard-00-01.abcde.mongodb.net.
```

### Step 4 - Restart your app

Well, you probably know best about that specific step.

## Q&A 

## Do I have to restart my app? 

Yes. The MongoDB driver uses the SRV protocol just for the first connection with the cluster. After it's connected with the cluster, it gets the data about the cluster from the cluster itself (that's how it knows who is the Primary, for example). So for the changes to take effect, you'll need to restart your app. 

## Am I free to use whichever domain I want? 

No. Technically, SRV allows you to do it, but since MongoDB uses TLS connection and SSL certifications are limited, the SRV records must have the same parent domain as the URI host. For example, if one of your hosts name is `oldcluster-shard-00-03.abcde.mongodb.net`, the URI domain must be `*.abcde.mongodb.net`

## Will it work indefinitely? 

Probably not. The SRV protocol has the list of current hostnames of the cluster nodes. When an instance fails and is replaced, it usually has a new hostname, which will make your SRV record outdated (unless you're using the same hostnames for the new replica).

## Conclusion 

DNS migration between MongoDB cluster relies on TXT and SRV records, and can help you with switching between clusters. However, keep in mind that the SRV records needs to be updated constantly with changes in the topology of the cluster, so it's better to use this solution as an intermidary step, and not as a fixed solution. 
