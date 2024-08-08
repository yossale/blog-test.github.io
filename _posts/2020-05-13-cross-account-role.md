---
title: Automating Cross-Account Role creation to access users’ account
description: >-
  Automating the creation of cross-account roles for access to your user’s account
image: cross-account-role.jpg
date: '2020-05-13'
author: 'Yossi Ittach'
authorName: 'yossi'
minutes: 7
---

Operating within users’ accounts is a serious matter. We want to make sure that we get all the credentials we need to deliver great user experience, but limit our access only to what the user feels comfortable with, and make sure there’s nothing we can do that we do not plan on doing. All this should happen automatically and seamlessly for the user. How do we get that done?

To be able to do that we need to answer 2 questions:

1. How do we get the relevant credentials and keys to our user’s account?

1. How do we operate within our user’s account once we have the above credentials?

## Getting relevant access to our user’s account (Technical Background)

The most basic way is to just ask your users to:

1. Create a role for you with the required credentials

1. Create a user in their system for you with the above roles, and limit the user to the above role

1. Generate an access key for that user

1. Manually send you the access credentials for the user they’ve created for you

This is, of course, error-prone in so many ways that it’s basically useless.

What we want is an automated process that will create an appropriate role in our user’s account, allow the user to review them, and limit the access only to us. That can be achieved using a [cross-account IAM role](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html).

To do that, you have to:

1. Send your *account number* and a *unique customer external ID* to the customer so they’ll be able to configure your access rights

1. Ask your customer to create a role with your required permission and attach the necessary policies to that role

1. Have your customer send you back the ARN for the role they created for you

This is much better than the previous solution, but it’s still very error-prone and manual. So what we need is a way to automate that process.

So now we get to the gist of our solution: How to automate the process of creating a dedicated role for our service at the user’s account and get the credentials, with as little human intervention as possible.

## Automating Cross account roles creation

This part is based on a very recommended in-depth, multiple-part AWS discussion about this specific practice ([1](https://aws.amazon.com/blogs/apn/easing-the-creation-of-cross-account-roles-for-customers/), [2](https://aws.amazon.com/blogs/apn/generating-custom-aws-cloudformation-templates-with-lambda-to-create-cross-account-roles/), [3](https://aws.amazon.com/blogs/apn/collecting-information-from-aws-cloudformation-resources-created-in-external-accounts-with-custom-resources/), [4](https://aws.amazon.com/blogs/apn/wrap-up-cross-account-role-onboarding-workflow/), [5](https://aws.amazon.com/blogs/apn/new-aws-cloudformation-stack-quick-create-links-further-simplify-customer-onboarding/)), but here we’ll try to keep it short and concise.

In overview, this is how the flow is going to look like:

1. A user comes into our website and enters her accountId

1. A Lambda is triggered which generates a specific CloudFormation template for this user

1. We create a link that will take the user directly to the AWS CloudFormation console, with the template already loaded, in her own account

1. The user clicks the “Create Stack” button to create the policy

1. As part of the execution of the stack, we get an automated SNS notification from AWS with the ARN of the role that was created.

1. We get the SNS message by using a Lambda to listen on the SNS topic we created for these notifications

1. We’re done!

## What do we need

The things we’re going to need are:

1. A CloudFormation template that generates the role we need with our policies

1. A publicly readable s3 bucket where the said template will reside

1. An SNS topic to which the stack execution results will be sent

1. A Lambda to customize the template per user

1. A Lambda to intercept the SNS message and update our database

### *Important:*
> *AWS SNS currently doesn’t support cross-region messaging. This means that if your topic is created in us-east-1, and the user runs her stack in eu-west-1, you won’t get the message. Since IAM roles are global, it doesn’t matter where your user is running the stack.*
> *For the sake of this example, we assume that your SNS and Lambda function, as well as the user’s stack run, occur in the same region. We’ll discuss possible solutions to this issue in future posts*

### 1. The CloudFormation Template

Let’s use a basic CloudFormation template with minimal EC2 capabilities.

```json
{
    "Parameters" : {
        "TrustedAccount" : {
            "Type" : "String",
            "Description" : "Your account id, to be trusted by the user"
        },
        "ExternalId" : {
            "Type" : "String",
            "Description" : "Your secret customer unique id"
        },
        "SnsArn" : {
            "Type" : "String",
            "Description" : "The ARN of the SNS topic you are listening on"
        }
    },
    "Resources": {
        "CrossAccountRole": {
            "Properties": {
                "AssumeRolePolicyDocument": {
                    "Statement": [
                        {
                            "Action": "sts:AssumeRole",
                            "Effect": "Allow",
                            "Principal": {
                                "AWS": { "Fn::Sub": "arn:aws:iam::${TrustedAccount}:root" }

                            },
                            "Condition": {
                                "StringEquals": {
                                    "sts:ExternalId": { "Ref": "ExternalId" }
                                }
                            },
                            "Sid": ""
                        }
                    ],
                    "Version": "2012-10-17"
                },
                "Path": "/",
                "Policies": [
                    {
                        "PolicyDocument": {
                            "Statement": [
                                {
                                    "Action": [
                                        "ec2:Describe*"
                                    ],
                                    "Effect": "Allow",
                                    "Resource": "*"
                                }
                            ],
                            "Version": "2012-10-17"
                        },
                        "PolicyName": "CustomerAccountAccess"
                    }
                ]
            },
            "Type": "AWS::IAM::Role"
        },
        "PhoneHomeCustomResource": {
            "Properties": {
                "ServiceToken": { "Ref": "SnsArn" },
                "RoleArn": { "Fn::GetAtt": ["CrossAccountRole", "Arn"] },
                "AccountID": { "Ref": "AWS::AccountId" },
                "ExternalID": { "Ref": "ExternalId" }
            },
            "Type": "Custom::PhoneHomeCustomResource",
            "Version": "1.0"
        }
    }
}
```

There are 3 main parts to this template:

1. **Parameters** is where we define the parameters we’ll update when customizing the template for each user

1. The **CrossAccountRole** object is responsible for describing who can access it (AssumeRolePolicyDocument) and what it can do (Policies).

1. The **PhoneHomeCustomResource** object basically tells AWS what to send back to your (in this case, the AccountId of the user who's running the template, the ARN of the role we created and the ExternalID), and where to send it to (ServiceToken). Any property we would add to the Properties object would be sent with the message.

### 2. The Bucket

That’s pretty straight forward: Take the template.json file and put it in a bucket, with public read access. Make a note of the *objectUrl* of the file, we'll use it later

### 3. An SNS Topic

Go to AWS -> SNS and create a topic called customer-resources-topic. Note the topic's ARN, we'll use it later.

### 4. The CustomizeUserTemplate Lambda

Our Lambda’s job seems pretty basic: For each user, generate a unique externalId token, and return to the user a url that will redirect her directly to the CloudFormation console, with the correct parameters.

AWS provides a direct way to do it, and the url looks like this: (This is not a broken link :) ) 

[https://console.aws.amazon.com/cloudformation/home?region=**region**#/stacks/create/review?stackName=**stack_name**&templateURL=**template_location**&param_ExternalId=**external_id**&param_TrustedAccount=**trusted_account**&param_SnsArn=**sns_arn](https://console.aws.amazon.com/cloudformation/home?region=**region**#/stacks/create/review?stackName=**stack_name**&templateURL=**template_location**&param_ExternalId=**external_id**&param_TrustedAccount=**trusted_account**&param_SnsArn=**sns_arn**)**

Let’s quickly get over them all:

* **region** — where the stack is to be deployed. Make sure this is the same region as your SNS, or your message won’t arrive

* **stack_name** — The name the user stack will have. Preferably something like “<OUR_COOL_COMPANY>_role” — It should be indicative enough so the user will understand what this stack is doing in her environment.

* **template_location** — the objectUrl of the template we placed in the bucket

* **external_id** — The unique user id you created for the user

* **trusted_account** — The account id of the account from which you’ll execute the role

* **sns_arn** — The ARN of your sns topic

And this is our Lambda:

```typescript
const cuid = require('cuid')
const AWS = require('aws-sdk')

// Add these to your lambda's environment variables
const REGION = 'us-east-1'
const SNS_ARN = process.env.SNS_ARN
const TEMPLATE_URL = process.env.TEMPLATE_URL
const TRUSTED_ACCOUNT = process.env.TRUSTED_ACCOUNT

var SNS = new AWS.SNS();

async function addPermissionToSns(userAccountId) {
  const snsPermissionRequest = {
    TopicArn: SNS_ARN,
    AWSAccountId: [userAccountId],
    ActionName: ['Publish'],
    Label: `AddCustomerPermission-${userAccountId}`,
  }
  await SNS.addPermission(snsPermissionRequest).promise()
}

function generateStackUrl(userExternalId) {

  const baseUrl = new URL('https://console.aws.amazon.com/cloudformation/home')
  baseUrl.searchParams.append('region', REGION)
  baseUrl.hash = ('/stacks/create/review')

  //These are not really query params: they are passed to the client and use the same annotation
  const searchParams = new URLSearchParams()
  searchParams.append('stackName', 'CoolCompany-Role')
  searchParams.append('templateURL', TEMPLATE_URL)
  searchParams.append('param_ExternalId', userExternalId)
  searchParams.append('param_TrustedAccount', TRUSTED_ACCOUNT)
  searchParams.append('param_SnsArn', SNS_ARN)

  return `${baseUrl.href}?${searchParams}`
}

module.exports.handler = async (event, context) => {

  console.log("Generating user template")

  const userAccountId = event.queryStringParameters.userAccountId
  const externalId = cuid()
  await addPermissionToSns(userAccountId)

  let generatedUrl = generateStackUrl(externalId)
  console.log(`Generated Url: ${generatedUrl}`)

  return {
    statusCode: 200,
    requestUrl: generatedUrl
  }
}
```

Basically, that (should have been) all. But the experienced AWS devs among you have probably already recognized a problem: We created an SNS topic and we want messages sent from a customer account sent to it. This means that we need to authorize the customer account to write to our SNS topic!

So here is our new Lambda:

```typescript
const cuid = require('cuid')
const AWS = require('aws-sdk')

// Add these to your lambda's environment variables
const REGION = 'us-east-1'
const SNS_ARN = process.env.SNS_ARN
const TEMPLATE_URL = process.env.TEMPLATE_URL
const TRUSTED_ACCOUNT = process.env.TRUSTED_ACCOUNT

var SNS = new AWS.SNS();

async function addPermissionToSns(userAccountId) {
  const snsPermissionRequest = {
    TopicArn: SNS_ARN,
    AWSAccountId: [userAccountId],
    ActionName: ['Publish'],
    Label: `AddCustomerPermission-${userAccountId}`,
  }
  await SNS.addPermission(snsPermissionRequest).promise()
}

function generateStackUrl(userExternalId) {

  const baseUrl = new URL('https://console.aws.amazon.com/cloudformation/home')
  baseUrl.searchParams.append('region', REGION)
  baseUrl.hash = ('/stacks/create/review')

  //These are not really query params: they are passed to the client and use the same annotation
  const searchParams = new URLSearchParams()
  searchParams.append('stackName', 'CoolCompany-Role')
  searchParams.append('templateURL', TEMPLATE_URL)
  searchParams.append('param_ExternalId', userExternalId)
  searchParams.append('param_TrustedAccount', TRUSTED_ACCOUNT)
  searchParams.append('param_SnsArn', SNS_ARN)

  return `${baseUrl.href}?${searchParams}`
}

module.exports.handler = async (event, context) => {

  console.log("Generating user template")

  const userAccountId = event.queryStringParameters.userAccountId
  const externalId = cuid()
  await addPermissionToSns(userAccountId)

  let generatedUrl = generateStackUrl(externalId)
  console.log(`Generated Url: ${generatedUrl}`)

  return {
    statusCode: 200,
    requestUrl: generatedUrl
  }
}
```

But, not so fast. If you want your Lambda to be able to really allow access to SNS topic to other users, the Lambda’s role should have that permission! So go to your Lambda -> permissions -> click on the role, and ‘add inline policy’:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AddCustomerAccountPermissions",
            "Effect": "Allow",
            "Action": "sns:AddPermission",
            "Resource": "<YOUR SNS ARN>"
        }
    ]
}
```

Now we’re done :)

### 5. The RegisterUser Lambda

This Lambda function’s job is to register in our database the ARN of the role that was created for us at the user account and the ExternalId. This Lambda needs to be triggered by our SNS topic, and this is how an update message looks like:

```json
{
  "Records": [
    {
      "EventSource": "aws:sns",
      "EventVersion": "1.0",
      "EventSubscriptionArn": "arn:aws:sns:us-east-1:111111111111:cross-account-sns:d507e8c5-3fa3-4d40-8e06-862b962fd57b",
      "Sns": {
        "Type": "Notification",
        "MessageId": "26b92bf2-6923-55da-b6cf-7d5a8848ab0d",
        "TopicArn": "arn:aws:sns:us-east-1:111111111111:cross-account-sns",
        "Subject": "AWS CloudFormation custom resource request",
        "Message": "{\"RequestType\":\"Create\",\"ServiceToken\":\"arn:aws:sns:us-east-1:123415165762:cross-account-sns\",\"ResponseURL\":\"https://cloudformation-custom-resource-response-useast1.s3.amazonaws.com/arn...\",\"StackId\":\"arn:aws:cloudformation:us-east-1:222222222222:stack/test/5ba240b0-6141-11ea-9a11-0e9711d18427\",\"RequestId\":\"a2b21671-3928-4dab-b991-2bb27c240179\",\"LogicalResourceId\":\"PhoneHomeCustomResource\",\"PhysicalResourceId\":\"2020/03/08/[$LATEST]4848484848488r8rjrjrj\",\"ResourceType\":\"Custom::PhoneHomeCustomResource\",\"ResourceProperties\":{\"ServiceToken\":\"arn:aws:sns:us-east-1:123415165762:cross-account-sns\",\"AccountID\":\"222222222222\",\"RoleArn\":\"arn:aws:iam::222222222222:role/test-CrossAccountRole-1R7TIJN9P2TCM\"}}",
        "Timestamp": "2020-01-01T13:01:31.627Z",
        "SignatureVersion": "1",
        "Signature": "....",
        "SigningCertUrl": "....",
        "UnsubscribeUrl": "....",
        "MessageAttributes": {}
      }
    }
  ]
}
```

What do we need for it:

1. Connect the SNS topic to the Lambda, so it will be triggered by it

1. Parse the message and update our database (or where ever you keep it) with the AccountID, ExternalID RoleArn

1. Respond correctly to the CustomResource call. (more about this after the Lambda)

Here is our Lambda:

```typescript
const response = require('cfn-response-promise')
const util = require('util')

async function replyToResourceRequest(event, context) {
  return await response.send(event, context, response.SUCCESS, {});
}

async function saveUserDetailsToDb(AccountId, RoleArn, ExternalID) {
  console.log(`Persisting the user info to the database ${AccountId}, ${RoleArn}, ${ExternalID}`)
  // do some db things...
}

module.exports.handler = async (event, context) => {

  console.log(`Got SNS message: ${util.inspect(event, { depth: 10 })}`)
  const snsMessage = event.Records[0].Sns
  const resourceEvent = JSON.parse(snsMessage.Message)

  let AccountID, RoleArn, ExternalID
  ({ AccountID, RoleArn, ExternalID } = resourceEvent.ResourceProperties)

  await saveUserDetailsToDb(AccountID, RoleArn, ExternalID)
  await replyToResourceRequest(resourceEvent, context)

}
```

**What does it mean “Respond correctly to the CustomResource call”?**

CloudFormation resource is an interesting topic by itself, but what’s important to understand in our case is this: CustomResources are actually AWS’s way of enabling users to create external resources as part of a stack. When a stack is created (in this case, when a user runs the stack to create our role), we’ll get a ‘Create’ message. If the user will delete her stack, we’ll get a ‘Delete’ message — and the same goes for ‘Update’.

The crucial thing to understand is that this is part of the stack creation process. This means that AWS is waiting for us for the successful completion of the stack creation/update/delete process. If we won’t respond promptly to the request, the stack creation process will fail and our hard-earned role will be deleted.

## Conclusion:

This picture from the [AWS blog](https://aws.amazon.com/blogs/apn/wrap-up-cross-account-role-onboarding-workflow/) sums up everything nicely

![](https://cdn-images-1.medium.com/max/2000/0*ZuAgjyPMeqYtw6sC)

We created a Lambda function that receives a user’s account ID (1), generates a personalized template for her and a unique externalId, and redirects her directly to the CloudFormation console, with the template loaded from the bucket (4). When the user executes the stack (5), the stack creates the role we defined in our template, and send us back the roleArn and the additional data we required (6). We then save the RoleArn, AccountID and the externalId (7) we generated so we can later utilize it using AWS STS, which will allow us to operate seamlessly within the user account.
