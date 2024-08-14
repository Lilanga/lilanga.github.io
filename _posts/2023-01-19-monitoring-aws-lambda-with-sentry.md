---
title: Monitoring AWS lambda functions issues and performance with Sentry
date: 2023-01-19 22:00:00 +0800
categories: [Serverless, AWS, Monitoring, Lambda]
tags: [Logging, AWS, Serverless, Lambda]
render_with_liquid: true
image:
  path: /assets/img/posts/2023-01-19/cover.png
---

AWS lambda is an excellent platform for creating serverless functions. AWS has sophisticated tools for monitoring telemetry data. With configured alarms, you can notify supporting teams when systems encounter runtime issues.

AWS lambda is a good use case for high-velocity data processing operations and processing unpredictable fluctuating workloads. The challenge of such scenarios is alerting tech teams to be alerted on abnormal scenarios in real-time. Relevant teams must be alerted to take proper action to mitigate possible impacts before abnormal behaviors create impactful damage to data.

Any relevant runtime insights collected during such incidents, such as impactful code segments, runtime information, stack traces, and related git change-sets, help to address identified issues promptly and effectively.

## Sentry for monitoring lambda functions

There are a couple of tools providing such functionalities. Sentry is one of the tools I integrated with my lambda functions in the past, and it saved us from lots of troubles that occurred in our step-functions during near real-time Kafka stream processing.

When configured correctly, Sentry gives you valuable real-time information and can configure ChatOps with Slack and teams to inform relevant groups immediately. (Or incident management tools like Pagerduty). Sentry supports NodeJS, Python, and .NET Core language stacks for AWS lambda functions.

## Configuring NodeJS lambda function with Sentry

We can use Sentry serverless npm package to add monitoring support to our lambda function. When we wrap our lambda function handler with a Sentry-provided wrap handler, the tool can capture all the telemetry data in real-time. Let’s use the following lambda function code and add Sentry support.

```javascript
export const handler = async(event) => {
    const querystring = event.queryStringParameters;
    const n = querystring.n || 0;
    const count = counter(n);

    return {
        statusCode: 200,
        body: `Final count is ${count}`,
    };
};

const counter = (n) => {
    let count = 0;
    if (n > 500000000) {
        n = 5000000000;
    }

    for (let i = 0; i <= n; i++) {
        count += i;
    }

    return count;
};
```

If you did not do it already, we must create an npm package with `npm init`. Since we are using the npm package, we can zip the whole project folder and use it to create the lambda function. With AWS CLI, you can easily do it. There are proper ways to develop lambda functions, such as SST (Serverless Stack) framework, which gives you proper tooling and project structure. But here, we are creating a simple function; let's use the basic approach with zip and upload.

Let's install the Sentry serverless package and add the necessary bootstrapping code to our code.

> npm install @sentry/serverless

Import and add the init config and update your Sentry DSN. Register for a Sentry free version and create a project. From there, you can get the DSN string. To integrate it with my lambda, I am using an environment variable called **SENTRY_DSN.**

```javascript
const Sentry = require("@sentry/serverless");

Sentry.AWSLambda.init({
  dsn: process.env.SENTRY_DSN,
  tracesSampleRate: 1.0,
});
```

Last step is to wrap our handler with the Sentry’s wrap handler.

`exports.handler = Sentry.AWSLambda.wrapHandler(handler);`

Now Sentry will capture all the statistics, and you can configure monitoring and alerting rules with near real-time notifications integrated with your corporate tools. Following is the complete code for the lambda function.

Following is the completed code for Sentry enabled lambda.

```javascript
const Sentry = require("@sentry/serverless");

Sentry.AWSLambda.init({
  dsn: process.env.SENTRY_DSN,
  tracesSampleRate: 1.0,
});

const handler = async(event) => {
    const querystring = event.queryStringParameters;
    const n = querystring.n || 0;
    const count = counter(n);

    return {
        statusCode: 200,
        body: `Final count is ${count}`,
    };
};

const counter = (n) => {
    let count = 0;
    if (n > 500000000) {
        n = 5000000000;
    }

    for (let i = 0; i <= n; i++) {
        count += i;
    }

    return count;
};

exports.handler = Sentry.AWSLambda.wrapHandler(handler);
```

Now we can create a zip file with the content of our code folder and use that to create the lambda function.

> zip -r node-sentry.zip .

## Sentry in action

Following is a sample error event created with the code of our lambda function.

![Captured error event](/assets/img/posts/2023-01-19/captured-error-event.png)
_Captured error event_

This view gives developers a good set of information about runtime issues so they can address them effectively.

Sentry also provides a good set of performance statistics for AWS lambda functions.

![Collected runtime statistics](/assets/img/posts/2023-01-19/runtime-statistics.png)
_Collected runtime statistics_

There are plenty of tools similar to Sentry. I have hands-on experience using this tool for lambda functions, and it has been highly effective. I hope this quick write-up may help someone understand creating a monitoring toolset for lambda functions.
