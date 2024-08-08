---
title: 'MongoDB: Handling Bulk Deletions'
description: 'Efficiently Handling Bulk Deletions in MongoDB'
date: '2024-07-30 10:00:00'
author: [Yossale]
---

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> The views expressed in this post are my own and do not reflect the official policy or position of MongoDB, Inc. The information provided is based on my personal experience and is intended as general guidance. These guidelines are not a substitute for professional advice and should not be relied upon as such.
{: .prompt-tip }

<!-- markdownlint-restore -->

## Context

Modern databases are mainly write-oriented: "Data is gold", storage is getting cheaper and regulations in most cases require long data retention periods. That means that deleting large bulks of data is something non-trivial, that you need to prepare to. 

## Deletions on MongoDB

In a simplified manner ([for a less simplified version](https://kb.corp.mongodb.com/article/000019603/)), when you delete a record, WiredTiger (the MongoDB storage engine), marks the place this record holds in the file as "deleted" (tombstone), and keeps a list of all the reusable places in the file. Next time you write something to that same collection, it'll overwrite this place.  

Basically, deletions **are** writes. When you delete a lot of documents, you need to go and mark them as deleted, and need to remove them from all the relevant indexes. Additionally, these changes need to be replicated from the primary to the secondaries. 

Deleting large percentages of the data from a large collection basically equals writing that same amount in terms of IOPS, network, CPU, tickets, Oplog replication lag, and replication window: It might create a high load on your production cluster and slow down your writes and reads. 

## Best practices for large deletes

### Delete in batches

Because of the additional load the deletion might incur on your cluster, the first thing we need to understand is what rate of deletions will not impact your cluster performances. So instead of deleting everything at once, try to test size-increasing batches (50K, 100K...) and check which ones give you the best balance between deletes size and performance impact. You might consider also running the deletes on off-peak hours. 

### Add a delay between batches

Deletes, as writes, go to the primary. From there, they need to be propagated to the secondaries. Deleting huge amounts of documents in rapid bulks might create a growing oplog replication lag between the primary and the secondary - which might impact our write / update latency, and in extreme situations, might trigger [flow control](https://www.mongodb.com/docs/manual/replication/#replication-lag-and-flow-control). 

To prevent the oplog gap from growing, add a delay between batches. As a rule of thumb, if deleting the batch took 60 seconds, add a 60-second delay to allow the secondaries to perform the same operation.

### Use `deleteMany` with Write Concern `majority`

When using the [deleteMany](https://www.mongodb.com/docs/manual/reference/method/db.collection.deleteMany/) with `{w: "majority"}` you'll be mitigating some of the replication lag issues, because the operation will only be ack'ed after it was propagated to at least a majority of the nodes. 

## How to reclaim your storage space

As mentioned in the beginning, WiredTiger doesn't clear deleted documents, but marks their locations as deleted and writes over them. But what happens if you decide to get rid of a large amount of redundant data, or move it to some cold storage? 

To reclaim this space, you'd need to run the [compact](https://www.mongodb.com/docs/manual/reference/command/compact/) command.