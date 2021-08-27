---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/why-predeclare-metrics
author: [[Brian Brazil]] 
---
> The standard way to use metrics in Prometheus is to declare them at file level, before using them. Why?

# Why predeclare metrics?


The standard way to use metrics in Prometheus is to declare them at file level, before using them. Why?

If you look at a typical use of direct instrumentation with Prometheus, it will look something like:

REQUESTS = Counter('my\_requests\_total', 'Requests made')

def some\_function():
  REQUESTS.inc()

And not:

def some\_function():
  prometheus.inc("my\_requests\_total")

Why is the former the way client libraries are designed?

The first reason relates to what was previously covered in [Existential issues with metrics](https://www.robustperception.io/existential-issues-with-metrics), metrics that only some times exist are difficult to deal with in PromQL. By declaring the metric before your application uses it, the client library can initialise it to 0. This applies even if the Counter is never subsequently incremented. However this doesn't work for metrics with labels: the client library doesn't know what labels might come up, so you have to do it yourself.

The second reason is performance. In the former example `REQUESTS` is a direct reference to the Counter object that you want to manipulate. In the latter there's an extra step where the client library would have to look up a thread safe map to find the right Counter object, which is often more expensive than the actual increment!

A third reason is that with the latter design, multiple different files could share a metric. This could cause confusion if there's an accidental collision. Even if it's on purpose, that's a bit of a smell as a metric is meant to tell you about a particular piece of code in a particular file. If that code is spread across multiple files, that's an indication that either each file should have its own differently named metric or the metric should be moved up the call stack.

That extra line of code buys you a lot, and you get to include a metric description too to help people using your instrumentation!

_Want advice on how to best instrument your code? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
