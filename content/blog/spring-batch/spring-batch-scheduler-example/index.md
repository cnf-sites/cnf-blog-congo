---
title: "Spring Batch Scheduler Example"
summary: "A detailed step-by-step tutorial on how to use a scheduler to run Spring Batch jobs using Spring Boot and Maven."
url: /spring-batch-scheduler-example.html
date: 2019-06-01
lastmod: 2019-06-01
tags: ["posts", "spring", "spring batch"]
draft: true
---

In this post I'm going to show you how to run Spring Batch at a schedule.

You'll also see how you can start/stop a scheduled job.

Let's get started.

## How Do You Schedule a Job with Spring Batch?

There are several ways to schedule a job with Spring Batch:

* You can use a scheduling tool like for example [Quartz](http://www.quartz-scheduler.org/).
* You can use the [scheduling capabilities of Spring](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/integration.html#scheduling).
* You can use a software utility that comes with the OS like for example [Cron](https://en.wikipedia.org/wiki/Cron).
