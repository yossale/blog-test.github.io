---
title: Creating users on Serverless Aurora/Mysql
---

As part of our new deployment, we started testing AWS's Aurora Serverless - [which is basically RDS on demand](https://aws.amazon.com/rds/aurora/serverless/) to use from our lambda functions.

However, unlike what you might expect from the "serverless" part of the name, the db is still created within a VPN, and is only accessible from within that
VPN, or through the [AWS Data Api](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.html)

So, you can connect to your db in two ways:
1. Through a bastion in a public network in the VPN - which is very useful if you want to connect to it using tools like DataGrop or MySql workbench
2. Through the Data Api - when you're using it from you lambdas

To connect via the lambda, you'd need to have the db user credentials stored an AWS secret. What does it mean?

1. You've created you database. In the creation process you created user `db_admin` with pass `admin_password`
2. You've connected to your database (either via the RDS console query screen or via a tunnel through the bastion)
3. you've created a new limited user for your lambdas to use - `app_user` with password `app_password`

Now you want to use the data-api in the lambda. Where do you provide the credentials?


4. Go to AWS Secrets Manager, and create a new secret. Select secret type "Credentials for RDS database", and you'll be prompt to enter the user name and password (in our case, `app_user` and `app_password`), and select your db from the dbs list at the bottom of the screen.

5. After the secret has been created, you now have the secret ARN - that what will need for the lambda.

6. In your lambda, you can now run:

```js
const AWS = require('aws-sdk')

exports.handler = async (event) => {

    //The db cluster ARN
    const DB_ARN = "arn:aws:rds:eu-west-1:123456789:cluster:my-db-cluster";
    const SECRET_ARN = "arn:aws:secretsmanager:eu-west-1:123456789:secret:db/app_user";

    var rdsdataservice = new AWS.RDSDataService({ apiVersion: '2018-08-01' });

    var params = {
        awsSecretStoreArn: SECRET_ARN,
        dbClusterOrInstanceArn: DB_ARN,
        sqlStatements: 'SELECT * FROM users',
        database: 'mydb'
    };
    const res = await rdsdataservice.executeSql(params).promise();

    console.log(util.inspect(res, {depth: 10}))

    const response = {
        statusCode: 200,
        body: JSON.stringify("success"),
    };
    return response;
};
```

**Make sure you create a policy to allow the lambda to access the DB and the secrets!**





