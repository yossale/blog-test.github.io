---
title: 'More than just event bus - EventBridge drill down'
description: "EventBridge might be based on an event bus, but it's becoming a better way to connect across accounts and services. Let's drill in."
date: '2020-12-09 10:00:00'
author: [Yossale]
---

A few months ago we discussed [which AWS service you could use to deploy a Pub-Sub solution](aws-pub-sub?utm_source=blog&utm_medium=link) and mentioned EventBridge as a leading candidate. But EventBridge capabilities make it more than just another event bus, and we'd like to drill down these options.

## EventBridge integration with external services

Modern-day applications rely on a lot of external services for monitoring, billing, authentication, etc. In most of these cases, you would register to these services and provide them with an endpoint to receive updates and handle them (like logins, rejected transactions, and others). AWS recognized it and decided to make it easier for developers to connect with these services more seamlessly.

For example, a lot of companies use external services like Auth0 to manage user identities and logins. One common caveat is the first-login: whenever a user signs for the first time, you usually want to run a unique process - create the user profile, send the user a "Welcome" email, and probably notify your customer success department.

Today, it would usually require an API gateway to receive requests from Auth0, validating them, and directing them to an SNS topic, which in turn will trigger the required functions.
![auth0-before](assets/images/2020-12-09-eventbridge-drill-down/auth0-before.png)

With an EventBridge connection, you can skip the API gateway, the authentication, and the SNS topic, and just add a rule to trigger the relevant lambdas:
![auth0-after](assets/images/2020-12-09-eventbridge-drill-down/auth0-after.png)

Unfortunately, this great solution is only available with a small number of AWS partners, so you can't rely on it too much, but if you happen to work with any of these partners, you'll be getting an easy out-of-the-box integration.

## Cross-account connections

The 3rd party integration feature of EventBridge is a nice wrapping over the basic EventBridge feature of cross-account connection.

As organizations rely more and more on multiple AWS accounts to separate their products, environments, and organizational units, the need for a simpler cross-account interaction has become more and more evident. Instead of building API gateways, running validations, and create additional buses, you can simply use EventBridge buses with policies to grant other service access to these buses.

Recently AWS also added a finer level of permissions handling, so you can:
- Allow another account to add only specific messages to your bus.
- Allow other accounts to add rules to your bus.

[This AWS blog post](https://aws.amazon.com/blogs/compute/simplifying-cross-account-access-with-amazon-eventbridge-resource-policies/) explains in detail how you can use these capabilities to create a 3-way cross-account messaging.

You can also read [this twitter-thread](https://twitter.com/jeremy_daly/status/1334238720199979008) by Jeremy Daly on common use cases for cross-account EventBridge buses.

## EventBridge as an Event Bus

Underneath all the 3rd parties connections and the fine-tuned policies, EventBridge is a Pub-Sub event bus (it's based on CloudWatch, which is AWS's cross-system monitoring tool). I've discussed its benefits as a Pub-Sub mechanism in an older post(https://yossale.com/aws-pub-sub), but there are additional features worth mentioning:

### Payload filtering
The standard way to send messages on event buses is to wrap them in an "envelope". This "envelope" contains all kinds of meta-data about the message itself, like sender, type, protocol version - and the message payload, which is the actual message.

So the "user.created" message
```json
{"userName": "John Doe", "email": "john.doe@gmail.com"}
```
Will be wrapped and look like this:
```json
{
  "message-type": "create-user",
  "version": "1",
  "payload": {"userName": "John Doe", "email": "john.doe@gmail.com", }
}
```
Most Pub-Sub services, like SNS, allows you to filter the destination based on the envelope data - something like "accept only messages from type A and protocol version 2".

EventBridge, on the other hand, allows you to filter messages also based on their payload - the message itself.
This is a cool trick, but it does have its caveats: First, the message has to be JSON, which is not always the case. Second, basing your messaging rules or filters on the content of the message might couple you tightly to implementation details that might change without notice.

### Data transformation
Another nice feature from EventBridge is the ability to transform the message before you send it. So if your rule got the message:
```json
{
  "message-type": "create-user",
  "version": "1",
  "payload": {"userName": "John Doe", "email": "john.doe@gmail.com", "address": "123 5th ave NY, NY"}
}
```
and you're forwarding to an email service that doesn't need the physical address, you can just define a new message template:
```json
{
  "name" : "$.payload.userName",
  "email" : "$.payload.email"
}
```
and the message that your target will receive will be:
```json
{
  "name" : "John Doe",
  "email" : "john.doe@gmail.com"
}
```
[You can read more about it here](https://docs.aws.amazon.com/eventbridge/latest/userguide/transform-input.html)
## Other EventBridge features

### The Schema Registry

At this point, you've probably already realized that EventBridge is aptly named "Bridge" because it connects different domains - different accounts, different services, and different vendors.

One of the most common problems when working with so many stakeholders is the integrations: each one has it's own event schema, and finding out these schemas is an annoying, labor-intensive task.

So AWS decided that every vendor (including themselves) that publishes to EventBridge will be able to register their schemas to [a central searchable location](https://docs.aws.amazon.com/eventbridge/latest/userguide/eventbridge-schemas.html#eventbridge-schemas-create), so whenever you want to use any schema, it'll be easily accessible.

You can find there all the AWS services schemas, the AWS Partners schemas, and upload your own (to use within your organization). You can also use the Schema Discovery tool, which will automatically analyze any incoming event to the event bus and add its schema to the registry.

AWS even threw in an option to download each schema as code binding in Java, Python, and TypeScript, so you'll get autocomplete and type validation when you code!

### What is the `default` event bus?

Every AWS account has the `default` event bus, even if EventBridge is never used. The reason it's there is that EventBridge is built on top of CloudWatch, and they both share their APIs, so all the CloudWatch events still need to go somewhere - and that "somewhere" is the `default` event bus. This means that the `default` event bus contains all the events emitted from AWS services in your account.

If you want to listen to internal account events, like S3 bucket creation, Lambda function triggering, or any other AWS service event, you'll need to set the rule on the `default` bus, as it states in the [documentation](https://docs.aws.amazon.com/eventbridge/latest/userguide/create-eventbridge-rule.html):

> "If you want this rule to trigger on matching events that come from your AWS account, select AWS default event bus. When an AWS service in your account emits an event, it always goes to your account’s default event bus"

## Should I switch to EventBridge?

EventBridge's capacity to create frictionless connections with external service providers and between accounts is a real upgrade - certainly when it comes to ease of use. Still, it doesn't render your current solutions obsolete.

If you're planning on consuming events from any of the AWS service partners, using EventBridge should be your go-to solution, and will make your dev-life easier. This is also the case if you're planning on cross-account messaging.

However, if you're already using an existing Pub-Sub solution like SNS, and not planning on using any of the unique EventBridge features, switching to EventBridge is probably not going to give you dramatic benefits.

EventBridge capabilities might not be big enough to entail a revamp of your current architecture, but it's a solution worth considering for future developments.


