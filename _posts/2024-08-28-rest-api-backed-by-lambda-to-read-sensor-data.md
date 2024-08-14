---
title: AWS - A REST API backed by Lambda and API Gateway using AWS CDK to Access Sensor Data from DynamoDB
date: 2024-08-28 06:00:00 +1000
categories: [AWS, Lambda, CDK, API Gateway, DynamoDB]
tags: [AWS, API-Gateway, CDK, Serverless, Lambda]
render_with_liquid: true
mermaid: true
image:
  path: /assets/img/posts/2024-08-28/cover.png
---


## Introduction

In the world of IoT, managing and accessing data from various sensors efficiently is crucial. AWS provides a powerful suite of tools that allow developers to seamlessly integrate IoT devices, store data, and expose this data via RESTful APIs.

In this post, we will walk through how to create a REST API using AWS CDK (Cloud Development Kit) in TypeScript to retrieve the latest sensor readings from a DynamoDB table. This example assumes you are working with sensor data stored in DynamoDB by AWS IoT Core, as we discussed in the previous blog post.

By the end of this tutorial, you will have a working AWS Lambda function that queries a DynamoDB table and an API Gateway endpoint to access this data using a REST endpoint.

## Why This Approach?

There are couple of benefits we can achieve using the approach we are discussing here. Namely, as follows.

**Serverless Architecture**: By using AWS Lambda, you create a scalable solution that only incurs costs when the function is invoked.

**Efficient Data Access**: DynamoDB is ideal for storing time-series data, such as sensor readings. This setup allows quick access to the most recent data.

**Infrastructure as Code**: AWS CDK enables you to define your cloud resources in TypeScript, ensuring your infrastructure is version-controlled and easily replicable.

## Setting Up the AWS CDK Project

I am hoping you have an AWS account with configured AWS CLI locally so you can use AWS CDK development kit effectively.

Let's start by setting up AWS CDK project for the solution.

```bash
mkdir sensor-api && cd sensor-api
cdk init app --language typescript
```

This command initializes a new CDK project in TypeScript.

Next step is to Install necessary AWS CDK libraries.

```bash
npm install aws-cdk-lib constructs
```

#### Create the Lambda Function

In the project directory, lets create a folder called `weather_lambda` and inside it, create a file named `weather_data.js`. Then we can add the following code to query the DynamoDB table and return the latest sensor reading.

```javascript
const { DynamoDBClient } = require("@aws-sdk/client-dynamodb");
const { DynamoDBDocumentClient, QueryCommand } = require("@aws-sdk/lib-dynamodb");

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

exports.handler = async (event) => {
    const params = {
    TableName: process.env.TABLE_NAME,
    KeyConditionExpression: "device_id = :deviceId",
    ExpressionAttributeValues: {
        ":deviceId": process.env.DEVICE_ID,
    },
    ScanIndexForward: false,
    Limit: 1,
    };

    try {
    const command = new QueryCommand(params);
    const data = await docClient.send(command);

    if (data.Items.length === 0) {
        return {
        statusCode: 404,
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ message: 'No data found' }),
        };
    }

    const latestReading = data.Items[0];

    return {
        statusCode: 200,
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
        device_id: latestReading.device_id,
        timestamp: latestReading.timestamp,
        humidity: latestReading.humidity,
        pressure: latestReading.pressure,
        temperature: latestReading.temperature,
        }),
    };
    } catch (error) {
    console.error(error);
    return {
        statusCode: 500,
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ message: 'Failed to fetch data from DynamoDB' }),
    };
    }
};
```

Here we are reading the latest sensor data record from the DynamoDB table. This table name is provided as environment variable of lambda function. We are expecting the same DynamoDB table structure we used in our previous sensor-data recording blog post.

#### Update the CDK Stack to suit our project

Now we can define the stack information for this AWS CDK project. Here, we need to create our Lambda function using defined lambda function code, provide necessary permission for our lambda to query DynamoDB table, and finally, create API gateway with an endpoint to access lambda function.

Open the `lib/sensor-api-stack.ts` file and replace its content with the following:

```typescript
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as apigateway from 'aws-cdk-lib/aws-apigateway';
import * as iam from 'aws-cdk-lib/aws-iam';
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';

export class SensorApiStack extends cdk.Stack {
    constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const weatherDataFunction = new lambda.Function(this, 'WeatherDataFunction', {
        runtime: lambda.Runtime.NODEJS_20_X,
        code: lambda.Code.fromAsset('weather_lambda'),
        handler: 'weather_data.handler',
        environment: {
        DEVICE_ID: 'DHT-22-01',
        TABLE_NAME: 'DHT22Data',
        }
    });

    const existingTable = dynamodb.Table.fromTableArn(this, 'DHT22Data', 
        'arn:aws:dynamodb:ap-southeast-2:ACCOUNT_ID:table/DHT22Data'
    );
    
    existingTable.grantReadData(weatherDataFunction);

    weatherDataFunction.addToRolePolicy(new iam.PolicyStatement({
        actions: ['dynamodb:Query'],
        resources: ['arn:aws:dynamodb:ap-southeast-2:ACCOUNT_ID:table/DHT22Data'],
    }));

    const api = new apigateway.LambdaRestApi(this, 'weatherAPI', {
        handler: weatherDataFunction,
        proxy: false
    });

    const weatherResource = api.root.addResource('weather');
    weatherResource.addMethod('GET');
    }
}
```

This stack file does the following three things:

1. Creates a Lambda function (`weatherDataFunction`) using the code we wrote earlier.
2. Grants the Lambda function read permissions to the existing DynamoDB table `DHT22Data`.
3. Sets up an API Gateway REST API to expose the Lambda function.

#### Deploy the Stack

Now we have everything in place. If we have AWS CLI configured and logged in correctly, we can deploy changes.

Lets deploy the stack using the following command:

 ```bash
cdk deploy
```

This will create the necessary AWS resources and give you an API endpoint URL.

### Accessing the Data

Once the deployment is complete, you can access the latest sensor reading by sending a GET request to the API Gateway endpoint:

```url
https://your-api-id.execute-api.ap-southeast-2.amazonaws.com/prod/weather
```

This will return the most recent sensor data, including humidity, pressure, and temperature.

## Conclusion

In this blog post, we walked through creating a REST API using AWS CDK to retrieve the latest sensor readings from a DynamoDB table. By leveraging AWS services such as Lambda and API Gateway, we built a scalable, serverless architecture that efficiently accesses and serves data. You can find the final solution at [this GitHub repo](https://github.com/Lilanga/cdk-api-with-lambda-apigateway). This approach not only simplifies the process of managing IoT data but also ensures that the infrastructure is easily manageable and replicable.

Feel free to expand on this setup by adding authentication, integrating with additional services, or handling more complex queries. AWS CDK offers a versatile platform for building robust cloud applications with minimal overhead.
