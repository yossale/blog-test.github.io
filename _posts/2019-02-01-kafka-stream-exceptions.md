---
title: 'Kafka Streams runtime exception handling'
---

# How to handle runtime exceptions in Kafka Streams

We've been using Kafka Streams (1.1.0, Java) as the backbone of our μ-services architecture. We've switched to stream mainly because we wanted the `exactly-once` processing guarantee. 

Lately we've been having several runtime exceptions that killed the entire stream library and our μ-service. 

So the main question was - is this the way to go? 
After a few back and forth, we realized that the best way to test this is by checking:
1. What would Kafka do?
2. Do we still keep our `exactly-once` promise?

## What **does** Kafka do?

[This document is the Kafka Stream Architecture design](https://cwiki.apache.org/confluence/display/KAFKA/Kafka+Streams+Architecture). After the description of the `StreamThread`  , `StreamTask` and `StandbyTask`, there's a discussion about Exceptions handling, the gist of which is as follows:

> First, we can distinguish between recoverable and fatal exceptions. Recoverable exception should be handled internally and never bubble out to the user. For fatal exceptions, Kafka Streams is doomed to fail and cannot start/continue to process data. [...] We should never try to handle any fatal exceptions but clean up and shutdown

So, if Kafka threw a Throwable at us, it basically means that the library is doomed to fail, and won't be able to process data. In our case, since the entire app is built around Kafka, this means killing the entire μ-service, and letting the deployment mechanism just re-deploy another one automatically.  


## Do we still keep our `exactly-once` promise?

Now we're faced with the question whether or not this behaviour might harm our hard-earned `exactly-once` guarantee.  
To answer that question, we first need to understand when the `exactly-once` is applicable. 

`exactly-once` is applicable from the moment we're inside the stream - meaning, our message arrived at the first topic, T1. So everything that happens before that is irrelevant: the producer who pushed the message to T1 in the first time could have failed just before sending it, and the message will never arrive (so not even `at-least-once` is valid) - so this is something we probably need to handle, but that doesn't have anything to do with streams. 

Now, let's say our message, M, is already inside topic T1. Hooray!

Now we can either fail before reading it, while processing it, and after pushing it. 
* If we failed **before** reading it, we're fine. The μ-service will go up again, will use the same appId, and we'll read the message.
* If we read it and failed before we even started processing it, we'll never send the offset commit, so again, we're fine.
* If we failed **during** processing it, again, we'll never reach the point of updating the offsets (because we commit the processed message together with the consumer offset - so if one didn't happen, neither did the other)
* If we failed **after** sending it - again, we're fine: even if we didn't get the ack, both the consumer offset and the new transformed/processed message are out. 

## Uncaught Exception Handlers

Kafka has 2 (non-overlapping) ways to handle uncaught exceptions:

* `KafkaStreams::setUncaughtExceptionHandler` - this function allows you to register an uncaught exception handler, but it will not prevent the stream from dying: it's only there to allow you to add behaviour to your app in case such an exception indeed happens. This is a good way to inform the rest of your app that it needs to shut itself down / send message somewhere. 

* `ProductionExceptionHandler` - You can implement this class and add it via the properties: `StreamsConfig.DEFAULT_PRODUCTION_EXCEPTION_HANDLER_CLASS_CONFIG` - but in this case, you will need to decide if the stream can keep going or not, and I feel this requires **very** deep understanding of the internals of the streams, and I'm still not sure when exactly you would want that.


## Conclusion

For us, using k8s deployments, with `n` number of pods of each service being automatically scaled all the time, the best way to handle runtime/unchecked exceptions is to make sure our app goes down with the Kafka Stream library (using the `KafkaStreams::setUncaughtExceptionHandler`), and letting the deployment service take care of starting the app again. 
