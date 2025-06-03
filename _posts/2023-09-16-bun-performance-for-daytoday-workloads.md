---
title: Performance of Bun for day-to-day workloads - Comparison with NodeJS, Golang and Rust
date: 2023-09-17 22:00:00 +0800
categories: [Optimisations, Performance, Scalability]
tags: [nodejs, bun, golang, rust, performance, scalability]
render_with_liquid: true
mermaid: true
image:
  path: /assets/img/posts/2023-09-17/cover.png
---

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

[Bun](https://bun.sh), the drop-in replacement for NodeJS, is a swiss-knife toolset for Javascript/Typescript development, including package managers, bundlers, and test runners in a single application.

The Bun runtime is powered by [JavaScriptCore](https://developer.apple.com/documentation/javascriptcore) instead of NodeJS V8 runtime and written using [Zig](https://ziglang.org) language. Bun runtime is a highly optimized runtime compared to existing Javascript runtime engines. According to the official documentation, Bun is designed from scratch, targeting Speed, Elegant APIs for everyday tasks, and a Cohesive Developer experience.

Here, let's observe what performance improvement Bun can provide(if any) compared to NodeJS. Let's compare it with Golang and Rust, generally considered languages for creating highly optimized executables, to get a better idea of the overall performance they provide.

## Performance comparison between NodeJS, Bun, Golang, and Rust

I am creating a straightforward HTTP server implementation using NodeJS, Bun, Golang, and Rust. I am using standard libraries to create HTTP servers without relying on frameworks except Rust, which do not have built-in HTTP implementation. For Rust, I am using wrap with tokio to implement the HTTP server.

I am creating a simple HTTP server endpoint to iterate over a given count and calculate the sum of the iteration count. When we use proper iteration count, we can create a CPU load with reasonable memory allocation to simulate our generic day-to-day calculation tasks.

This performance comparison is done on my development machine without standard practices, so this cannot be treated as benchmarking performance between these languages. But This profiling will give us a better understanding of the optimizations and resource utilizations we can expect from these language stacks in our day-to-day development work.

I am using the following language versions in this test

- NodeJS: `v20.4.0`
- Bun: `1.0.0`
- Golang: `go1.21.1`
- Rust: `1.72.0`

### NodeJS API implementation

Following is the NodeJS implementation of the HTTP server

```javascript
const http = require('http');
var url = require('url');

const listner = (req, res) =>{
    const qs = url.parse(req.url, true).query;
    const iterations = (qs && qs.value||0);
    const count = counter(iterations);

    res.writeHead(200);
    res.end((`Final count is ${count}`));
};

const counter = (n) => {
    let count = 0;
    if (n > 5000000000) {
        n = 5000000000;
    }

    for (let i = 0; i <= n; i++) {
        count += i;
    }

    return count;
}

const server = http.createServer(listner);
server.listen(3000, ()=>{
    console.log(`Server is running on http://localhost:3000`);
});
```

### Bun API implementation

Following is the Bun implementation of the same logic. Most of the implementation is identical in comparison with NodeJS implementation. But here I am using Typescript, which Bun has drop-in support.

```typescript
const server = Bun.serve({
    port: 3000,
    fetch(req) {
        const url = new URL(req.url);
        const qs = url.searchParams;
        const iterations: number = +(qs.get('value')||0);

        const count = counter(iterations);
        return new Response(`Final count is ${count}`);
    },
});

const counter = (n: number) :number => {
    let count = 0;
    if (n > 5000000000) {
        n = 5000000000;
    }

    for (let i = 0; i <= n; i++) {
        count += i;
    }

    return count;
}

console.log(`Listing on http://localhost:${server.port}`)
```

### Golang API Implementation

Following is the Golang implementation of the same logic using standard libraries.

```go
package main

import (
  "fmt"
  "log"
  "net/http"
  "strconv"
)

func getCount(w http.ResponseWriter, r *http.Request) {
  stringCount := r.FormValue("value")
  iterations, _ := strconv.ParseInt(stringCount, 10, 64)
  count := counter(iterations)
  fmt.Fprintf(w, "Final count is %d", count)
}

func main() {
  mux := http.NewServeMux()
  mux.HandleFunc("/", getCount)
  log.Fatal(http.ListenAndServe(":3000", mux))
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

```

### Rust API Implementation

Following is the Rust implementation of the same logic. Here, we are using [`tokio`](https://tokio.rs/) and [`warp`](https://crates.io/crates/warp) libraries to create our HTTP server since Rust only has a tcp standard library.

```rust
use serde::{Serialize, Deserialize};
use std::collections::HashMap;
use warp::Filter;

#[derive(Serialize)]
pub struct GenericResponse {
    pub status: String,
    pub message: String,
}

#[derive(Serialize, Deserialize)]
struct QueryParams {
    value: String,
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
async fn main() {
    let counters = warp::get()
        .and(warp::query::<HashMap<String, String>>())
        .map(|p: HashMap<String, String>| match p.get("value") {
            Some(count) => Ok(format!("Final count is {}", counter(count.trim().parse::<u128>().unwrap())).into()),
            None => Ok(format!("No query string found")),
        });

    println!("🚀 Server started successfully");
    warp::serve(counters).run(([0, 0, 0, 0], 3000)).await;
}

```

## Profiling HTTP server implementaiton

Now, we have our server implementations. For Golang and Rust, I created optimized binaries with release configurations. For Bun, I created a server.js file using bun build command.

Let's run our HTTP servers and see what kind of Throughput these languages can provide. Here, I am using the bombardier load tool to generate the load. I am using 10000000 as the value, so our code will execute for 10000000 iterations for each request and yield the result. Our load tool will generate 25 requests for each endpoint with three concurrent requests at a given time.

Following is the command I used with [the bombardier](https://pkg.go.dev/github.com/codesenberg/bombardier).

`bombardier -c 3 -n 25 http://localhost:3000/\?value\=10000000`

### NodeJS HTTP server implementation profiling

Following are the results for the NodeJS runtime
Here, we can see one request out of 25 is timed out. Throughput is 459.92/s
![nodejs api profiling](/assets/img/posts/2023-09-17/nodejs-api-profiling.png)

### Bun HTTP server profiling

Following are the results for Bun runtime profiling
Here, all 25 requests are served successfully, and Throughput is 16.70KB/s

![bun-api-profiling](/assets/img/posts/2023-09-17/bun-api-profiling.png)

### Golang HTTP server profiling

Following are the results of Golang implementation profiling
All the requests are served successfully in Golang implementation as well. Throughput is recorded as 88.75KB/s

![golang-api-profiling](/assets/img/posts/2023-09-17/golang-api-profiling.png)

### Rust HTTP server profiling

Following are the results for the Rust implementation of the HTTP server logic.
All the requests are served successfully in the Rust implementation as well. The recorded Throughput of the server for these 25 requests is 1.79MB/s

![rust-api-profiling](/assets/img/posts/2023-09-17/rust-api-profiling.png)

## Summary of the profiling results

Following is the summary of profiling results collected by each implementation profiling.

| Langauge stack | Avg. Reqs/sec | Latency | Throughput | Success requests | Failded requests |
|--- | --- |--- |--- |--- |--- |
| NodeJS | 2.03  | 1.58s | 459.92/s | 24 | 1 |
| Bun | 88.2 | 37.48ms | 16.70KB/s | 25 | 0 |
| Golang | 578.74 | 6.93ms | 88.75KB/s | 25 | 0 |
| Rust | 8389.11 | 340.20µs | 1.79MB/s | 25 | 0 |

### Throughput comparison between NodeJS and Bun

<div>
  <canvas id="throughput"></canvas>
</div>

Bun is roughly 36 times faster compared to NodeJS. This is a significant improvement in throughput.

### Latency comparison between NodeJS and Bun

<div>
  <canvas id="latency"></canvas>
</div>

Bun is roughly 42 times faster compared to NodeJS. This is a significant improvement in latency.

## Conclusion

According to the statistics we collected, bun runtime significantly improved compared to NodeJS runtime. Here, we are using Node version 20, which is announced as having many runtime improvements compared to older versions of NodeJS.

Golang and Rust are still way faster than these JavaScript engines. It makes sense because those languages produce optimized compiled binaries while NodeJS and Bun use JavaScript interpretation with Just-In-Time compiling techniques to improve the code execution. However, Bun's underlying FTL (named Faster than Light) JIT compiler is faster than light compared to NodeJS.

So many factors contribute to optimized server resource utilization and optimal code executions. But comparing the same NodeJS and Bun Javascript runtime eco-systems, it looks like Bun can give a significant advantage for Javascript-based applications.

<script>
  const throughput = document.getElementById('throughput');
  const latency = document.getElementById('latency');
  new Chart(throughput, {
    type: 'bar',
    data: {
      labels: ['NodeJS', 'Bun'],
      datasets: [
      {
        label: 'throughput',
        data: [459.92, 16700],
        borderWidth: 1
      }]
    },
    options: {
      scales: {
        y: {
          beginAtZero: true
        }
      }
    }
  });

  new Chart(latency, {
    type: 'bar',
    data: {
      labels: ['NodeJS', 'Bun'],
      datasets: [
      {
        label: 'requests/sec',
        data: [2.03, 88.2],
        backgroundColor: 'rgba(255, 99, 132, 0.4)',
        borderWidth: 1
      }]
    },
    options: {
      scales: {
        y: {
          beginAtZero: true
        }
      }
    }
  });
</script>
