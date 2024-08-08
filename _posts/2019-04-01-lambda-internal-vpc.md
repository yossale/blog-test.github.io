---
title: Basic Lambda to call internal VPC api
---

Product team decided they wanted a specific event to happen every time a specific email address receives an email. The first option was to poll mail server and analyse all the received emails (yuck!). 

The other option was to use AWS tools. In our case, forward the email to `SES` and call a lambda that will trigger our internal service with all the email meta-data - Which is exactly what we did. 

## Few pointers before the code:

1. Lambda doesn't load node dpendencies - so if you want to use some external packages except aws, you'd need to upload a zip with the dependencies and your code
2. If you want to call an internal VPC resource, you need to:
   1. Assign the lambda to your VPC
   2. Assign the lambda a security group that will enable it to work from within the vpc

After you've assigned the Lambda to the VPC and setup the SG, the rest is easy:

```js
const axios = require('axios'); //Loaded in the zip file!

console.log(`Url: http://${process.env.SERVICE_URL}`)

exports.handler = (event, context, callback) => {
    console.log(`Inside lambda: ${JSON.stringify(event)}` )
    
    axios.post(`http://${process.env.SERVICE_URL}`, event)
        .then((res) => {
            console.log(res);
            callback(null, res);
        })
        .catch((error) => {
            console.error(error);
            callback(error);
        });
        
    console.log(`Logging out`)    
};

```

You can find all the event types here: [Sample Events](https://docs.aws.amazon.com/lambda/latest/dg/eventsources.html) 

> The SES -> Lambda invocation only send the Email's meta-data. If you want to have the email content, you'd need to use SNS (so SES -> SNS Topic -> Lambda), but bear in mind that SNS only supports emails up to 150K, so for anything larger, you'd need to move to S3
