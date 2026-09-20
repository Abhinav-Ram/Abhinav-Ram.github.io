---
title: "Performance Comparison between High-Performance Web Application Frameworks — A Case Study"
date: 2024-06-12
excerpt: "A practical comparison of FastAPI and Gin under increasing concurrency."
tags:
  - Python
  - Go
  - FastAPI
  - Gin
  - Benchmarking
  - Web Performance
---

Performance Comparison of web application frameworks is essential for analyzing user experience, scalability, reliability, and resource efficiency. The chosen frameworks for this particular case study are — FastAPI (Python) and Gin (Golang), with the reasons provided below.

---

## Introduction of Languages and Their Respective Frameworks

In this section, a brief overview of both the languages and their respective frameworks are provided.

### Python and FastAPI

Python is a high-level, interpreted language that is best known for being simple, highly readable and for its vast community support and ecosystem of libraries. Due to the same, it is the most popular language for Machine Learning and Data Science tasks.

FastAPI is a high-performance python-based application framework designed for building RESTful APIs in Python. It makes use of multiple components such as pydantic for data validation and serialization, Starlette which is an ASGI (Asynchronous Server Gateway Interface) to support concurrent requests, and also uses Uvicorn, a minimal low-level server/application interface which follows the ASGI specifications.

### Go and Gin

Go is a high-speed, statically typed, compiled language, which is famously known for the containerization platform built on it, Docker. It has an extremely fast compile time which is attributed to its efficiency in dependency analysis (and unused imports are an error, essentially eliminating the compile time for those). These features, make it one of the best languages for building high-performance backend apps, along with Rust, Kotlin, and of course, Python.

Gin is a web-framework written in Golang, and mainly focuses on developer productivity due to its minimalist approach to building web applications. It has its own built-in templating system, but doesn’t necessarily follow an MVC design pattern, so that is left up to the developer. Its features are flexibility in routing, middleware support, JSON rendering to build APIs, and overall extensibility. It also features a radix-tree implementation for extremely fast routing and it is 40 times faster than its counterpart, Martini.

Since FastAPI and Gin are two of the frameworks that strike a good balance between performance and developer productivity, they are chosen for this analysis.

---

## Benchmarking Methodology

Benchmarking is done in order to evaluate the performance and other capabilities of the web application frameworks in handling computationally intensive tasks such as scientific calculations, database retrieval and processing. The process involves simulating various trials by changing number of datapoints to be processed, number of concurrent users, etc., thereby evaluating key performance metrics.

### Performance Metrics

The following the chosen metrics:

- Response Times (Time taken to respond) — Lower times mean faster performance, crucial for user experience.
- Throughput (Requests per Second) — Higher throughput shows better scalability under load.
- Resource Utilization (CPU and Memory) — Efficient usage means better performance and cost-effectiveness.

### Tools

1. Locust: Used to simulate concurrent users making requests to the web applications — Provides detailed statistics on response times and throughput.
2. System Monitoring Tools:
    - pidstat: Used to monitor CPU and memory utilization of the web applications during the benchmarking process.
    - htop: Used to verify the overall system load and resource consumption.

### Test Scenarios
The benchmarking process involves several test scenarios designed to evaluate the performance of the web frameworks under different load conditions:

- **Endpoint:** `/fibonacci/{n}` calculates and returns the Fibonacci sequence up to the nth number.
- **Range of Fibonacci Numbers:** Fibonacci numbers for n = 10, 20, 30, 40, and 50 are calculated to test both frameworks under varying computational loads.
- **Number of Concurrent user Simulateds:** 50, 100, 200, and 500 concurrent users are simulated to evaluate how each framework handles increasing levels of concurrency.

---

## Test Environment

The above process was executed in the following environment:

**Hardware**

| Component | Specification |
|---|---|
| CPU | Intel Core i5-10300H @ 2.50 GHz |
| Cores / threads | 4 cores, 2 threads per core, 1 socket |
| RAM | 8.00 GB (7.84 GB usable) |

**Operating system:** Ubuntu 22.04.3 LTS (release 22.04)

**Software**

| Software | Version |
|---|---|
| Python | 3.10.12 |
| FastAPI | 0.111.0 |
| Uvicorn | 0.30.1 |
| Go | 1.22.4 (linux/amd64) |
| Gin | v1.10.0 |

---

## Benchmarking Results

All latencies are in milliseconds. "RPS" is the request rate Locust reported at the end of each run. Failed requests are included in the request counts.

### Results by load level

| Users | Framework | Requests | Failures | Median | Average | p95 | p99 | Max | RPS |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|
| 50 | FastAPI | 4,495 | 0 | 2 | 2.17 | 5 | 12 | 133 | 84.5 |
| 50 | Gin | 2,810 | 0 | 1 | 2.12 | 6 | 12 | 24 | 79.5 |
| 100 | FastAPI | 9,016 | 1 | 2 | 2.21 | 7 | 13 | 42 | 166.5 |
| 100 | Gin | 9,175 | 0 | 1 | 2.64 | 10 | 18 | 52 | 170.5 |
| 200 | FastAPI | 17,883 | 0 | 2 | 3.50 | 15 | 27 | 73 | 348.2 |
| 200 | Gin | 18,835 | 0 | 2 | 3.01 | 14 | 23 | 76 | 338.6 |
| 500 | FastAPI | 47,139 | 2 | 3 | 6.51 | 32 | 61 | 207 | 831.4 |
| 500 | Gin | 45,242 | 0 | 8 | 13.63 | 47 | 82 | 168 | 810.5 |

The 50-user Gin run recorded fewer total requests (2,810 vs 4,495) at a similar request rate, which suggests it ran for a shorter time, so compare rates and latencies at that level, not counts.

### Gin relative to FastAPI

A positive percentage means Gin was slower.

| Users | Average latency | p95 latency | Throughput (RPS) |
|---|---:|---:|---:|
| 50 | −2% | +20% | −6% |
| 100 | +19% | +43% | +2% |
| 200 | −14% | −7% | −3% |
| 500 | +109% | +47% | −3% |

### Average latency by endpoint at 500 users

| Endpoint | FastAPI | Gin |
|---|---:|---:|
| `/fibonacci/10` | 8.37 | 18.71 |
| `/fibonacci/20` | 6.25 | 11.87 |
| `/fibonacci/30` | 6.42 | 13.56 |
| `/fibonacci/40` | 5.98 | 12.07 |
| `/fibonacci/50` | 5.54 | 11.91 |

### The 500-user runs in detail

The full Locust reports for the 500-user runs add percentile distributions and failure details. The FastAPI report matches the Excel export above (47,175 vs 47,139 requests, with identical median, average, p95 and p99). **The Gin report does not match the Excel export**, and it shows much higher latency:

| 500 users | Requests | Failures | Median | Average | p95 | p99 | Max | RPS |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| FastAPI | 47,175 | 2 | 3 | 6.51 | 32 | 61 | 207 | 831.4 |
| Gin (Excel export) | 45,242 | 0 | 8 | 13.63 | 47 | 82 | 168 | 810.5 |
| Gin (full report) | 41,718 | 0 | 52 | 82.99 | 250 | 360 | 908 | 776.3 |

Aggregated latency percentiles from the full reports:

| 500 users | p50 | p60 | p70 | p80 | p90 | p95 | p99 | p100 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| FastAPI | 3 | 4 | 5 | 6 | 11 | 32 | 61 | 210 |
| Gin (full report) | 52 | 70 | 94 | 140 | 200 | 250 | 360 | 910 |

**Failures.** FastAPI had 3 failed requests out of 78,533 across all runs (0.004%). All three were on `/fibonacci/10`. The two at 500 users were connection-level errors (one `RemoteDisconnected: Remote end closed connection without response`, one `ConnectionResetError: Connection reset by peer`). Gin had no failures in any run.

<!-- TODO: add pidstat / htop CPU and memory figures for each run -->

---

## Interpretation

**1. Throughput was the same for both frameworks.** In every run, both frameworks delivered roughly 1.6 to 1.7 requests per second per simulated user, and their request rates stayed within about 6% of each other. That linear scaling suggests the load was set by the number of users (who pause between requests) and not by server capacity. Neither framework hit a throughput ceiling, so the real differences are in latency.

**2. Up to 200 users there is no meaningful winner.** Medians stayed at 1 to 2 ms and averages between 2 and 3.5 ms. Gin was slightly faster at 200 users, and FastAPI was slightly faster in the tail at 100 users. The gaps are a few milliseconds and would not be noticeable to a user.

**3. At 500 users, FastAPI came out ahead, which contradicts the usual expectation that Go wins.** In both Gin reports, Gin's median, average and tail latencies were higher than FastAPI's. The size of the gap, though, is not stable: Gin's average was 13.63 ms in one report and 82.99 ms in the other. The Locust charts show FastAPI's latency starting high during ramp-up (average of about 30 ms, p95 of about 70 ms) and then settling to roughly 9 to 10 ms once all 500 users were active. Gin's chart in the full report shows the average climbing to about 155 ms and only easing to about 120 ms by the end of the chart (values read from the charts, so approximate). Such a large difference between two Gin runs points to interference from the test setup, not to a stable property of the framework.

**4. The endpoint is too light to separate the frameworks on computation.** Latency did not rise from n = 10 to n = 50, even though responses grew nearly eight times in size, because building a sequence of at most 50 numbers is trivial next to HTTP handling and serialization. This benchmark therefore measures per-request framework overhead, not computational speed.

**5. `/fibonacci/10` was consistently the slowest endpoint.** This held for FastAPI and both Gin reports (for example, 103 ms vs 75 to 81 ms on the others in the full Gin report). All of FastAPI's failures were also on this endpoint, and they were connection resets and disconnects, not application errors. A likely explanation is connection setup and warm-up pressure while users are still being spawned, but the reports don't timestamp the failures, so this is a hypothesis.

**6. Reliability was very good for both.** FastAPI had 3 failures across 78,533 requests, and Gin had none across 76,062 (Excel export) or 41,718 (full 500-user report).

### Limitations

- Locust, the servers and the monitoring tools shared one 4-core, 8 GB machine. At 500 users (about 800 requests per second) the load generator itself competes with the server for CPU, which can inflate latency for whichever server is measured.
- Each configuration was run once, and the two Gin reports at 500 users disagree, so run-to-run variance is clearly not negligible.
- Only one lightweight endpoint was tested, so the results say nothing about CPU-heavy or database-bound workloads.

### Conclusion

For a lightweight endpoint at up to roughly 800 requests per second on this hardware, FastAPI and Gin performed comparably. Both were stable, with negligible errors. At 500 users FastAPI was faster in every report, but the Gin results varied too much between runs to attribute the difference to the frameworks themselves. This study does not show that either framework is faster in general.

A fairer comparison would use a CPU-heavy endpoint (for example, a recursive Fibonacci at n = 35 or higher), a separate machine for the load generator, and several repeated runs per configuration.