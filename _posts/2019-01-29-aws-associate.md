---
title: AWS Certified Solutions Architect - Associate
---

I've been doing a Udemy course as a preparation for the [AWS Certified Solutions Architect - Associate](https://www.udemy.com/aws-certified-solutions-architect-associate). These are my summary notes

# AWS Certified Solutions Architect - Associate

## Exam
* 130 minutes
* 60 questions
* Results are between 100 - 1000, pass: 720
* Scenario based qestions

## IAM

* Users
* Groups
* Roles
* Policis

Users are part of Groups
Resources have Roles : i.e, for an instance to connect to S3, it needs to have a role 
All the User groups and Roles get their permissions are through Policies, which are defined by json:

```
# God mode policy
{
    "Version":"2019-01-01",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        }
    ]
}
```

(Creating policies is not part of the exam)

### General
* IAM is cross-regional 
* "root account" is the account created with the first setup of the AWS account, and has complete Admin access.
* New users have no permissions until assigned
* New users are assinged Access Key Id and Secret Access Keys when created, for the api access. 



## S3

* Key - Value Object based, with metadata and versioning
* Has access control lists
* Max 5TB file size
* Buckets are universal namespace (https://s3-{region}.amazonaws.com/{bucket})

### Consistency model:
* Read After Write consistency - object will be availabe for readdirectly after being   written    
* Eventual consistency for overwrite PUTS and DELETES

### Storage Tiers/Classes
* S3 Standatrd - 99.99% availability, 99.999999999% durability, cross-devices, cross-facilities redundancy, designed to sustain loss of 2 facilities at the same time. 
* S3 - IA (infrequently access) - for data accessed less frequently. Lower storage fee, but has a retrieval fee. 
    S3 One Zone - IA : the same as IA, only in 1 AZ. (cheaper)
* Glacier: Very cheap, archival only. Standard retrieval time takes 3 - 5 hours.

### Cross Region Replication (CRR)
* Requires versioning enaled on the source bucket

## CloudFront 

* Edge Location - the location the content will be cached: Per AWS Region (They are not read only, you can write to them too, and they will replicate to the origin and from there to others)
* Clearing cache cost money :)
* Origin - The original file location: S3 bucket, EC2 instance, ELB, or Route53
* Distribution - all the locations of the Edges you defined
* Can distribute dynamic, static, streaming and interactive content (Web Distribution: most common, for websites; RTMP - media streaming)


## EC2 

### Placment groups

Two types: 
1. Cluster placment group - A group of instances within a **single** AZ that need low latency / high throughput (i.e cassandra cluster). Only available for specific types. 
2. Spread placment group - A group of instances that need to be place **seperatly** from each other

* Placment group name myst be unique within aws account
* Only available for certain instance types
* Recommended to use homogenous instances within placment group
* You can't merge placment groups
* You can't move an exisitng instance to a placment group, only create it into it 

## EFS
* Supports NFSv4 
* Only pay for used storage
* Scales up to petabytes
* Support thousands of concurrent NFS connections
* Data is stored across multiple AZ within region


## Route 53

### DNS overview
* NS - Name Server record. Meaning, if I go to `helloRoute53gurus.com`, and I'm the first one to try it in my ISP, then the ISP server will ask the .com if it has NS recored for helloRoute53gurus. 
The .com will have a record that maps it to `ns.awsdns.com`. So it'll go to `ns.awsdns.com` , which will direct it to Route53..\    
* A - short for Address - that's the most basic record, and it's the IP for the url
* TTL - time to live - how long to keep in cache
* CNAME - resolve one domain to another (can't be used for 'naked' domain names, e.g: 'www.google.com' )
* Alias - unique to Route53, the may resource records to Elastic Load Balancer, CloudFron, or S3 bucket that are configured as websites. They work like CNAME (www.example.com -> elb1234.elb.amazonaws.com)
*  MX record - email records
* PTR Records - reverse lookups

> ELB do not have predefined IPv4 addresses, you resolve them using a dns name. So basically, if you have the domain "example.com" and you want to direct it's traffic to an ELB, you need to use an Alias (not a cname, because it's a naked domain name!, and not an A record because it has no IP)  

### Routing Policies

* Simple Routing - 1 record with multiple ips addresses, randomly returned. No health checks
* Weighted Routing - 1 record with N% goes to one rcecord, and M% to another and so forth
* Latency Based Routing - Route 53 will send to the region with the lowest latency
* Failover Routing - Health check based Primary/Secondary routing: if the primary instance fails (health check = false), directs to the secondary
* Geolocation Routing - config which geo location goes to which instance
* Multivalue Answer Routing - Several records, each with ip addresses, and health check for each resource. The ips will return randomlly, so it's good for disparsing traffic to different resources. 

## VPC

* NAT Gateways - scaled up to 10G, no need to patch/add security groups/assign ip (automatic), they do need to be updates in the routing table (so they can go out via igw)
* Network ACL - 
    * It's like a SG, in the subnet level. 
    * Each subnet is associated with one, but default it's blocking all in/out bound traffic. you can associate multiple subnets to the same ACL, but only 1 ACL per subnet. 
    * The traffic rules are evaluated from the lowest value and up.
    * Unlike SG, opening port 80 for incoming will not allow outbound response on port 80. If you want to communicate on port 80, you'd have to define rule both for inbound and outbound. (Otherwise, it'll go in and not out)
    * You can block IP addresses using ACL, you can't with SG
* ALB - you need at least 2 public subnets for an Application Load Balancer


## Application Services

### SQS
* Distributed Pull based Messaging queue
* Up to 256 Kb messages
* Default retention: 4 days, max 14 days
* Default promisese "at-least-once", "FIFO" promises exactly once with ordering
* Can poll with timeout (like kafka)
* Visibility - once message is consumed, it's marked as "invisible" for 30 seconds (default, max is 12 hours), and if it's not marked as "read" within that time frame, it returns to be visible and re-distributed to another consumer. 

### SWF - Simple Workflow Service

* Kind on amazon ETL system, with Workers (who process jobs) and Deciders (who control the flow of jobs). The system enables dispatching of jobs to multiple workers (which makes it easily scalable), tracking the jobs status, and so forth. 
* SWF keeps track of all the tasks and events in an application (in SQS you'd have to do it manually)
* Unlike SQS, In SWF a task is assigned only once and never duplicated (What happens if the job fails? IDK). 
* SWF enables you to incorportae human interaction - like, if someone needs to approve received messages, for example

### SNS - Simple Notifications Services

Delivers notification too: 
* Push notifications
* SMS
* Email
* SQS queue
* Any http endpoint
* Lamda functions

* Messages are hosted in multiple AZ for redundancy

Messages are agregated by Topics, and recipients can dynamically subscribe to Topics.

### Elastic Transcoder

* Convert video files between formats - like formatiing video files to different formats for portable devices


### API Gateway

Basically a front API for your lamda/internal APIs, with amazon capabilities:

* API Caching - caching responses to an request api with TTL
* Throttling requests to prevent attacks

### Kinesis

3 types:
* Streams - Kafka (Retention : up to 7 days) - Shards = partitions (?) 
* Firehose - Fully automated, no consumers, No retention, No shards - Can be written to s3 / elastic 
* Analytics - run SQL queries on the streams/firehose streams, and write the result to s3 /elsastic
