---
title: 'Pub-Sub Services on AWS: What are your options?'
description: How to pick the best service to use for a Pub-Sub solution on AWS Infrastructure
date: '2020-08-05 10:00:00'

---
## Pub-Sub architecture

Pub-Sub architecture is a common way to create decoupled service-to-service communication between multiples services. It allows you to add, remove and alter services dynamically and message multiple services at the same time (you can read more about the Pub-Sub architecture [here](https://www.ably.io/concepts/pub-sub))

In the cloud, this pattern has an additional advantage: When you have multiple dynamically allocated resources, it's much easier to handle everything when all your service needs to know for it to work properly is a single broker address.

![Pub-Sub architecture](/assets/images/2020-08-05-aws-pub-sub/pub_sub_arch.png)

Instead of deploying your own Pub-Sub solutions, AWS has several managed services that can be used as your Pub-Sub backbone.

## What are your options?

AWS offers three primary services that can help you to implement Pub-Sub in your account:

* Kinesis
* SNS
* EventBridge

## Kinesis

Kinesis is AWS's version of Kafka. It's a very efficient, extremely scalable system for collecting, processing, and analyzing real-time streaming data.

Kinesis's capabilities will allow you to use it as a Pub-Sub mechanism, but this service was designed for very different purposes in mind (e.g streaming video) - which makes its infrastructure much more complex then is required for a Pub-Sub mechanism. Besides, it lacks the seamless connectivity with other AWS services (SQS queues, emails) that other services have - So we wouldn't recommend using it as a Pub-Sub backbone.

## SNS

SNS is AWS's "go-to" solution for Pub-Sub. It exists since 2010, can be triggered by 30 different AWS services (and of course, your custom triggers), and it can trigger not only Lambda functions but also emails, web addresses, SQS queues, SMS messages, and mobile push notifications. It can support millions of subscribers per topic.

SNS is a "push" service - whenever a message is sent to the topic, all the topic subscribers will immediately (avg latency of 25 msec) be triggered.
Additionally, SNS supports Message Attributes and subscription filtering - so for example, if for each message you publish to SNS you'll add a `type` attribute, you'll be able to use that to subscribe specific subscribers (Lambda function, SQS queue, email service) only to a certain type of messages - thus limiting the number of needless invocations, and automating the process.

SNS's real-time, multiple subscribers push broadcast makes it an ideal candidate for serving as a Pub-Sub backbone, but it does have its drawbacks: If you have a Lambda subscribed to an SNS, and you send 1000 SNS messages to the topic, then you'll have 1000 invocations of the Lambda function running at once. Such a high load has several implications:

First, it might tax your system heavily - external services which do not scale well might fail, or your DB might get overloaded and cause other services to slow down, for example.

Second, it might cause the throttling of your Lambda functions. Once you've reached your Lambda invocation limits (default 1000 per region), you might experience two different types of problem:
1. Some other, unrelated Lambdas in the same region will be throttled, so other services might experience failures.
2. Some of your messages might get lost because there would be no available Lambda functions to serve them.

Which brings us to the question of how SNS handles errors. SNS has a [default retry behavior for each type of endpoint](https://docs.aws.amazon.com/sns/latest/dg/sns-message-delivery-retries.html), and after these retry policies have been exhausted, the messages will be discarded. To prevent the discard of these messages, each SNS subscription can define a Dead Letter Queue (DLQ) where all discarded messages will be directed to for handling.

What about costs? You can read about SNS delivery costs [here](https://aws.amazon.com/sns/pricing/), but the nice thing about using SNS as a Pub-Sub backbone with Lambda functions and SQS queue as subscribers is that there's no charge for deliveries to SQS queues and Lambda functions.

## EventBridge

EventBridge is AWS's new shiny version of CloudWatch Events. Unlike SNS, which originated as a notification service to multiple protocols (HTTP, SMS, push notifications, emails, etc), EventBridge is pre-designed as an Event Bus - and thus have more event handling oriented options. Its basic architecture is similar to SNS: EventBridge has Buses that you can publish to, Destinations that will receive the messages, and Rules to filter the messages per destination.

Since it's designed as an Event Bus, EventBridge does not support direct publish to different protocols and is limited to 5 Destinations per Rule. On the plus side, it can publish directly to 17 (and growing) AWS destinations, supports getting messages from external software services (like Zendesk, for example) directly into your system, and allows you to define message schemas and do basic message transformations.

Message transformation lets you send one message with several attributes to the bus, and transform the message such that each service will receive a subset of the message body. For example, You can send a message with the CC number, but make sure that only the billing service will receive the full message, while other services will only receive the non-financial attributes.

## So, which one should you use?

As always, the answer is "It depends". If you have numerous subscribers and latency is an issue, or if you need to publish multiple messages/emails - stick with SNS. If you're building an Event-Driven architecture with a limited amount of services and you need to control the schema structure and run a basic transformation on the schema - use EventBridge

In our next posts, we'll drill on each service and how it can be used build a Pub-Sub architecture.
