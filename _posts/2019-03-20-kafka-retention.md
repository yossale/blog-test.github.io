---
title: Some kafka retention lessons (we learned the hard way)
---


We're using Kafka (2.0 on cluster, Java client 1.1) as our messaging backbone. A different cluster is deployed in each environment, and we recently started seeing some weird behaviour in one of our user acceptance environments.

## First Problem: Old messages are suddenly reprocessed. 

Once in every few version releases, we suddently saw some services re-processing old messages. After a lot of head banging and tedious head scratching, we found out that in Kafka, the retention on the offsets and the retention on the messages is not necessarily the same. 

What does it mean? Well, when a messsage is sent to a specific Kafka topic, it's retained as long as the topic retention. So, if our topic retention is 1 week, then after 1 week it will no longer be available. 

The consumer offset, however, is a different story. The consumers offsets are saved in an internal topic called `__consumer_offsets`, and their retention time is defined in the parameter `offsets.retention.minutes` in the [broker config](https://kafka.apache.org/documentation/#brokerconfigs) with a deafult of 24 hours. 

So what happened to us is this: Our messages retention was set to 2 weeks, and the offsets retention was 24 hours. After a period of not using the system, we deployed a new version. Once the new version was up, it queried the Kafka topic for it's latest offset. However, the `consumer_offset` of this application id was already deleted, and the default behaviour is to read from the begining of the stream - which is exactly what happened to us: This is why we were consuming old messages when we released new versions, and it would only happen if we released versions after more than 24 hours and less than 2 weeks. 

## Second Problem: The producer attempted to use a producer id which is not currently assigned to its transactional id

This one was even more annoying. We're using Kafka streams api, which promises an exactly-once message processing. Every once in a while, we'd get the above error *after* the message has been proccessed. This would cause the Kafka stream to shut down, the app to restart - and then, to process the same message again(!). 

Now, this was extremly weird. First of all, it was a violation of our "exactly-once" constraint. In addition, we had no idea what it means!

Lately we started also seeing what seems to be a related error: 
`org.apache.kafka.common.errors.UnknownProducerIdException: Found no record of producerId=16003 on the broker. It is possible that the last message with the producerId=16003 has been removed due to hitting the retention limit.`

So we started thinking - what is happening to our producer ids? 

Then we discovered this: 

[transactional.id.expiration.ms](https://docs.confluent.io/current/installation/configuration/broker-configs.html#transactional-id-expiration-ms)

>The maximum amount of time in ms that the transaction coordinator will wait before proactively expire a producer’s transactional ID without receiving any transaction status updates from it.

> Type: int

> Default: 604800000

604800000 ms is 7 days. So basically, if we've had a streaming application that had no traffic for 7 days, it's producer metadata was deleted - and that's the behaviour we've been seeing: the application consumed the message, processed it - and when it tried to commit the transaction and update the offset, it failed. This is why we processed, crushed, and re-processed. 

## Bottom line

Kafka is a tool built for massive data streaming, and it's defaults are organized around it. Both these issued occured because this specific environment's usage pattern is random and is not corresponding with the cdefault configuration. 
