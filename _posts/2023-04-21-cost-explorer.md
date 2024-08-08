---
title: 'Finding AWS Service Names for the Cost Explorer Command'
description: How to get AWS services names for Cost Explorer queries.
date: '2023-04-21 10:00:00'
author: [Yossale]
---

AWS Cost Explorer can be a powerful tool for monitoring your AWS spending, but it's not always easy to use. One common issue is finding the correct AWS service names for the cost-and-usage command. For example, the service name for S3 is not "S3" or "Amazon S3" - it's "Amazon Simple Storage Service". Additionally, if you use the wrong service name, you'll get a valid response with a monthly sum of 0, which can be frustrating.

## The Solution: Using the get-dimension-values Command
To find the correct service names for the cost-and-usage command, you can use the get-dimension-values command. Here's an example of how to find the service name for DynamoDB:

Use AWS CLI or Cloud Shell and run the following command:

```sh
aws ce get-dimension-values --search-string Dynamo --time-period Start=2023-01-01,End=2023-03-31 --dimension SERVICE
```

This command will return a list of all the possible values for the SERVICE dimension that contain the word "Dynamo".

Look for the service name that matches the AWS service you want to monitor. For DynamoDB, the service name is "Amazon DynamoDB".

Use the service name in the cost-and-usage command to get the monthly cost for the service. For example, to get the monthly cost for DynamoDB from the beginning of the year, you can run:

```sh
aws ce get-cost-and-usage --time-period Start=2023-01-01,End=2023-03-31 --granularity MONTHLY --metrics "UnblendedCost" --dimension "SERVICE=Amazon DynamoDB"
```

You can use this same process for any AWS service you need to monitor. Just replace "Dynamo" with a keyword related to the service you're looking for.

