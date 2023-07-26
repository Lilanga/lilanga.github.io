---
title: AWS Lambda resources utilisation comparison using NodeJS, Golang and Rust
date: 2023-01-18 20:30:00 +0800
categories: [Serverless, AWS]
tags: [NodeJS, Golang, Rust, AWS]
render_with_liquid: true
image:
  path: /assets/img/posts/2023-01-18/cover.png
---

AWS lambda is one of the popular serverless computing service that is well-suited for many functional/event-based processing requirements.

Even in managed computation platforms like cloud-based Kubernetes clusters (like EKS), we have to involve in system maintenance, capacity planning, buffer resource allocations, planning and configuring for smooth automatic scaling, managing platform security, patching platforms, etc.

The beauty of a function-based computation platforms like AWS lambda is you do not need to worry about any of the platform-level operational activities, and you are paying only for the exact amount of resources you used for your computation activity.

As we are getting billed for the execution duration and the consumed resources (memory) rather than computation units like EC2 instances/Worker nodes, Correct computation runtimes will be the key to optimizing the cost. This becomes essential when running high-velocity workloads such as processing streams of IoT payloads, creating a significant difference in monthly billing.

If CPU demand is low, we can use a cheaper arm-based computational stack, and when memory usage is low, it will cost us less money for function executions.

## Assessing resource utilisation for different language stacks

Let’s use Golang, Rust, and Nodejs languages to assess the performance. These language stacks are popular and considered optimized runtimes nowadays.

I am creating three lambda functions with the exactly same logic implementation using those three languages. Later, let’s execute these functions using a load generator tool to asses the memory, CPU utilization and average function-initialize durations. To collect these matrices, I enabled Cloud watch lambda insights. Using a cloud watch dashboard, we will be able to see our matrices in a graphical representation.

I am using my favorite bombardier load generator tool to generate the required load for cloud functions at the same time. Please note I am not targeting to measure runtime optimisation of those languages such as concurrency, scalability, or efficiency with CPU-intensive computational workloads. However, These optimisations will indeed will contribute to our CPU and memory utilisation metrices. Our primary concern is how effective the tech stack for high velocy functional executions and which one will be highly cost effective.

Functions will be called with 75 iterations of computation tasks which will emulate the average data processing workload of IoT data packet processing.

Following Lambda functions accept an argument with a query string parameter which is used as iterations count for the calculation.

## NodeJS function

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

## Golang function

```golang
package main

import (
 "context"
 "fmt"
 "strconv"

 "github.com/aws/aws-lambda-go/events"
 "github.com/aws/aws-lambda-go/lambda"
)

func Handler(ctx context.Context, request events.APIGatewayProxyRequest) (string, error) {

 num := request.QueryStringParameters["n"]
 n, _ := strconv.ParseInt(num, 10, 64)
 return fmt.Sprintf("Final count is %d", counter(n)), nil
}

func counter(n int64) int64 {
 count := int64(0)

 if n > 5000000000 {
  n = 5000000000
 }

 for i := int64(0); i <= n; i++ {
  count += i
 }

 return count
}
func main() {
 lambda.Start(Handler)
}
```

## Rust function

```rust
use lambda_http::{run, service_fn, Body, Error, Request, RequestExt, Response};

async fn function_handler(event: Request) -> Result<Response<Body>, Error> {
    let n = event.query_string_parameters()
    .first("n")
    .unwrap_or("0").to_string();

    let resp = Response::builder()
        .status(200)
        .header("content-type", "text/html")
        .body(format!("Final count is {}", counter(n.parse::<u128>().unwrap())).into())
        .map_err(Box::new)?;
    Ok(resp)
}

fn counter(n: u128) ->u128{
    let mut count = 0;
    let mut i = 0;
    let mut range = n;

    if range>5000000000 { range = 5000000000; }

    while i <= range {
        count += i;
        i+=1;
    }

    return count;
}

#[tokio::main]
async fn main() -> Result<(), Error> {
    run(service_fn(function_handler)).await
}
```

All these aws lambda functions will be created with lambda function urls so we can use `bombardier` to generate load.

## Lambda function resource utilisation

When functions getting executed Lambda insights capture various telemetry data including resources utilisation.

Following are the analytics captured by CloudWatch Lambda insights for each lambda function. We are using CloudWatch dashboard to display statistics side by side.

![lambda resources utilisation](/assets/img/posts/2023-01-18/resource-utilisation.png)
_lambda resource utilisation_

NodeJS consumes the highest level of resources, while Golang consumes low resources. However, Rust uses the best-optimized resource utilization comparing the three languages.

With the stats, we can see Rust will have the least execution cost per function with the least amount of CPU and memory allocations per function. Function initialization duration is also lowest in Rust.

So with this kind of workload, Rust will give you the highest level of cost optimization. This can be varied with concurrent operations and high CPU-intensive workload, which is not measured here.

But overall, this will help us get the idea about adopting the tech stack for our cloud functions while considering other external factors.
