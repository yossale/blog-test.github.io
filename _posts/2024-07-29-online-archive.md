---
title: 'MongoDB Atlas: Online Archive - BKM and common pitfalls'
description: 'MongoDB Atlas Online Archive: Expert Tips and Common Challenges'
date: '2024-07-29 10:00:00'
author: [Yossale]

---

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> The views expressed in this post are my own and do not reflect the official policy or position of MongoDB, Inc. The information provided is based on my personal experience and is intended as general guidance. These guidelines are not a substitute for professional advice and should not be relied upon as such.
{: .prompt-tip }

<!-- markdownlint-restore -->

The [MongoDB Atlas Online Archive](https://www.mongodb.com/docs/atlas/online-archive/manage-online-archive/) (OA) - is a great solution for data tiering when using MongoDB, and a lot of customers are using it. As it gets more widely spread, I wanted to write a short overview of the common BKM and pitfalls I've seen at customers, and hopefully help others to better plan their usage and avoid the cul-de-sacs.

- This is not an introductory post: RTFM.

## Context

A lot of customers need to save a lot of hardly used past data accessible for either regulatory purposes or because it's still being used seldomly. It creates an unnecessary load on their infrastructure, resulting in larger indexes, larger disks, and higher bills. Online Archive provides them with the option to use "colder" storage without using another infrastructure with complicated ETL, with no code changes.

## Why use it?

- Enables you to significantly reduce your cluster size by moving tons of data to the OA - smaller disks and smaller indexes result in lower tiers, which translates to reduced expenses.
- Complex aggregations on large amounts of data will run much faster - The OA data is saved in parquet files, and supports columnar structure, so large "OLAP-ish" analytics queries might run even better than your cluster.
- Easy to run - the archiving process is managed by MongoDB, so you don't need to do anything.
- No code changes required: It uses the same query language, so there's no difference between accessing the MongoDB cluster or the online archive except the connection string you're using.
- You can have a connection string that will connect you to either the cluster itself, the online archive, or both.
- Cheap ( ~$5 for each TB processed)
- When querying based on the defined partitions, it's quite fast for a cold storage!

## Limitations

- This is an archive. You can not change data in the archive.
- You can not delete a specific record (yet) - So GDPR might be an issue
- If you're using the date criteria - i.e., choose a specific date field and define after how many days you want it moved to the OA - you can only change the number of days, not the field itself. So if there was any logical change, you're stuck with it.
- You can not use a computed value in the Custom Criteria option.
- You can not merge two Online Archives into one (You can use Data Federation for that, see below)
- You can only have **one** active OA on a collection. You can pause one and start another one, but they can not have the same partitioning structure.
- Doesn't support UUID as an **optimized** partition field (so don't use the UUID as the partition field, the search will take hours)

## I want to start using Online Archive, what should I do

- Use the custom criteria for archiving. It's more cumbersome, but you'll be able to edit it later in case you need to.

- Online Archive partition supports all [these types](https://www.mongodb.com/docs/atlas/data-federation/supported-unsupported/supported-partition-attributes/#supported-partition-attribute-types) but only optimizes on `ObjectId`, `string`, `boolean`, `date`, `int`, `double` and `long`. For these types, it creates index files to speed up the queries it serves. If you use a supported but not optimized type (like UUID, for example) you might get very high latency on queries for that field.

- The OA archive job runs every 5 minutes, deleting up to 2GB per run, and can be set to run only on pre-defined time frames. So if you plan to archive TB of data, keep in mind that it will take several days to archive the data - and it will add additional load on your cluster.

## Getting over common problems

### I don't understand how to write the custom criteria
 The syntax is the [`$expr`](https://www.mongodb.com/docs/manual/reference/operator/query/expr/) operator, so it looks like this:

```json
 {
    "$expr": {
        "$eq": [
            "$status",
            "CLOSED"
        ]
    }
}
```

### Can I use the date part of the `ObjectId` as a `createdAt` field if I don't have one?
Technically, you can - see the code. However, this will not use the index, so you'll be doing a full scan on all the `_ids` in the system every time the archiving job runs, so **don't**.
```json
 { //Copy-Paste protection: This will have terrible performance implications. Don't use it.
        "$expr": {
            "$lte": [
                {"$toDate": "$_id"},
                { "$subtract": [ "$$NOW", 2592000000 ] }
           ]
        }
 }
```

### My partition field is a UUID
 UUID can be used as a partition field, but it's not optimized, so the search will take forever and you won't be happy. If you have no other choice, add a `uuidStr` field to your document, as a string representation of your UUID, and partition (also) by that field.

### I want to delete my cluster but I want to keep my Online Archive
 You can't. Use the federated db and `$merge` all your data from the Online Archive to an S3 bucket, so you can still have access to that data (albeit slower) via the federated db connection string.

### I need to merge 2 (or more) Online Archives
You can't merge Online Archives. What you can do is create a Federated Databases and place these two Online Archives under the same db/collection - whichever suits your needs.

### I've archived tons of data but no disk space has been released, wtf?
WiredTiger, the MongoDB storage engine, doesn't release evacuated space within the collection, it just marks it as cleared and will re-write on it when additional data is added. If you want to release the cleared space, you'll need to run the `compact` command.

### I need to update a document in the archive 
You can't change a document in the archive, but you can rewrite it to your cluster - and then when it's archived, it will run over the existing document in the archive - if they have the same `_id`

### I need to get things back from the OA to my cluster
This process is called "rehydration". You basically want to `merge` data from the OA into the cluster - so your best way to do it is connect both the OA and the cluster under a federated db, and then run a [`$merge`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/merge/) aggregation from the OA into the relevant collection.
