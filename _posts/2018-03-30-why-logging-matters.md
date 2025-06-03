---
title: Why logging matters for software engineers
date: 2018-03-30 15:00:00 +0530
categories: [Logging, Best practices]
tags: [Logging, DevOps]
render_with_liquid: true
image:
  path: /assets/img/posts/2018-03-30/cover.png
---

Logging the processing statuses was a common practice from the early stages of computing. Since computers run various subroutines and combinations of different execution tasks to conduct a particular processing, it is essential to maintain a sequential log to analyze the execution order and determine the failing scenarios if anything goes wrong. Useful logs will save lots of time for developers and application engineers. Proper logs will provide tremendous support when doing application troubleshooting or supporting production incidents.

Apart from our developed solutions, third-party applications and devices maintain processing logs for the same reason. For example, the Engine Control Unit (ECU) in your car even keeps a journal of activities it performs along with logs of any abnormal activities and conditions it may encounter. It provides the essential information to troubleshoot any failures in the latter.

When it comes to systems that consist of different components, logging plays a vital role by providing the proper evidence of the communication journey for issue investigations and performance improvements, which cannot be achieved only using Application Performance Monitoring (APM) or any other type of real-time monitoring mechanism.

## Importance of log analysis

![Importance of log analysis](/assets/img/posts/2018-03-30/importance-of-log-analysis.png)
_Importance of log analysis_

Log retention for extended periods is not a problem for most cases when logging happens at the correct format and velocity. Historical logs provide valuable insights to find out abnormal behaviors of the system, pain points of executions, and optimization points for enhancements. Mining those logs using the proper tools enables developers to uncover performance bottlenecks and improvement points which not visible during the development phase. Scanning logs for extended periods provide rich clues, especially when isolating root causes for sporadic issues. In addition, scanning logs for extended periods enable us to identify security loopholes and abnormal behaviors causing security back-doors.

## Different kinds of logs used in software engineering

Mainly, we can generate two types of logs in software engineering.

- Diagnostic Logging
- Audit Logging

**Diagnostic logging** is done to support failure investigations and identify abnormal behaviors. Diagnostic logging should be created carefully to cover critical functional areas and modules with proper message structure to give the process state context while avoiding extensive logging with too much information. These types of logs should generate when necessary, and most of the time, logging operations should be executed on specific code segments or pathways related to abnormal execution paths or exception behaviors.

**Audit logging** is done for business requirements. The idea is to support auditing various actions performed on systems so that interested parties can take actions and decisions based on those audit trails. These types of logs record business transactions or activities that happen in real-time from the point of business perspective.

Audit logs are essential for fulfilling certain types of legal requirements. In some instances, these audit logs help identify failed transactions and when restoring them to their original status. Moreover, valuable data captured in these audit logs help do statistics and business analysis using data mining techniques to get various insights about the business.

## Synchronous vs. Asynchronous logging

There are mainly two types of approaches to generating logs. Writing logs using asynchronous way or do it using synchronous methods. Both approaches have their advantages and disadvantages. We should select the correct approach based on our needs.

**Synchronous Logging:** In synchronous log generation, the application gets blocked until the log is written to the log. This approach guarantees the log entry is committed successfully before moving on to the following operation. This approach ensures that logged messages are written to the logs in the correct order, and logging operations' reliability is very high too. For that reason, the synchronous approach is encouraged for audit logging.

When an application generates a high velocity of logs frequently or IO operations are heavy for log writing, it is better to use asynchronous loggers since synchronous loggers may slow down the application significantly.

**Asynchronous Logging:** Asynchronous logging can improve application processing by offloading log generation work to a separate thread. This method will give applications a significant performance boost compared to the synchronous approach if the application generates lots of log entries.

Suppose the application generates high-velocity logs and frequent log updates due to high throughput. In that case, asynchronous logging offers huge benefits over synchronous logging approaches. Moreover, this method has the lowest logging response time latency.

However, error handling is weak when we use asynchronous loggers, and retry cycles must be implemented to ensure the operation's reliability. In such cases, we cannot guarantee the order of the logged information. Suppose an application is logging faster than the asynchronous logger's capability. In that case, queues will fill up, and some messages will likely be lost. Because of these reasons, asynchronous logging is not recommended for audit logs.

## Using proper levels while logging

All standard logging libraries support logging levels, and it is essential to use proper log levels when we are logging something. Appropriate log levels will significantly help identify the criticality of events and enable various actions based on those levels, including some automated monitoring actions. Basic log levels are specified as “error, warning, information, debug, trace” based on their criticality in descending order.

**Error:** We can use the error level to log errors with a customer impact, causing problems with the system's proper functionality. These types of errors need immediate attention. Usually, the on-call roaster should be called upon for some of these types of failures. We can use this log level if something goes wrong and we need proper attention to fix it.

**Warn:**  An unexpected technical issue that can be potentially harmful. But does not need immediate attention or human innervation. Supporting engineers need to focus on identifying the failure or risk these types of issues can cause in current or future contexts.

**Info:** Things need attention if they happen in large volume or high velocity. It may not be a faulty situation, but we need to pay attention to identify any abnormal behaviors.

Unusual scenarios that can be problematic if happening with high frequency can be logged under this category. Session resets, high-volume repeated database calls, and login failures due to wrong credentials are a few examples that we need to pay attention to if frequently happening with high volume.

**Debug:** Any information that can be used to track business flows and can be helpful to identify faulty situations when doing issue investigations can be logged under this category. These logs are specially targeted to aid when isolating malfunctioned behaviors in Development and Quality Assurance phases.

Debug logs usually contain entry and exit points of methods, functions, and procedures, including decision points inside those methods.

**Trace:** Trace log entries create highly detailed information such as full execution stack, dumps of allocated objects, etc. These types of logs are expected only to enable in development environments and produce a detailed, high volume of entries to the logs.

## Distributed Logging Mechanisms

![Distributed logging](/assets/img/posts/2018-03-30/distributed-logging.png)
_Distributed logging_

When developing large enterprise applications, different components are hosted on different servers, and the same components are hosted on server farms containing fleets of servers.

If logs generated by each of those servers are retained with those specific servers, when it comes to debugging issues, inspecting all those logs for possible related errors is time-consuming and, in most cases, not practical.

A better solution to this problem is building a centralized logging application to collect various types of logs and aggregate them in a centralized place.

Moreover, with the evolution of micro-services-driven applications and cloud-hosted scalable application environment setups, traditional log handling mechanisms became obsolete and needed a richer unified approach to handle and maintain logs.

To address these problems, we are using distributed logging mechanisms.

There are four main steps for distributed logging systems.

- **Collecting Logs:** Logs are collected from different environments and different locations
- **Log Transportation:** Transport logs to a centralized place
- **Storage:** Scalable storage mechanism with log rotation, archiving, and access rights.
- **Log analysis:** Provide various tools and UI components to easily analyze the stored log information in a convenient way.

Optionally we can configure automated alerting based on log analysis. Alerting is useful when managing hundreds of servers, which cannot easily pin down abnormal behaviors due to data load.
