---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/how-does-a-prometheus-gauge-work
author: [[Brian Brazil]] 
---
# How does a Prometheus Gauge work?

> We looked previously at the counter, how does the Prometheus gauge work?

---
We looked previously at the [counter](https://www.robustperception.io/how-does-a-prometheus-counter-work/), how does the Prometheus gauge work?

The Prometheus gauge is essentially the same simple idea as gauges in other monitoring systems. Gauges can go up and down over time, and scrapes take a snapshot of the current value. There's no potential for messing around with moving averages or resets, as there is with counters.

Where Prometheus differs from other other monitoring systems is the API for gauges in instrumentation. The [Prometheus Gauge API](https://prometheus.io/docs/instrumenting/writing_clientlibs/#gauge) covers all the gauge uses cases in one place. It has `inc`, `dec`, and `set` methods, plus various utility functions for common uses such as setting the current time. The client library takes care of all the bookkeeping and thread safety for you. There's usually also some way to do a callback such as `set_function` in Python, `setChild` in Java, and Go has `GaugeFunc`.

By contrast the popular [Dropwizard](http://metrics.dropwizard.io/3.1.0/manual/core/) metric library offers a number of different gauge classes, primarily based on what processing you'd like it to perform before exposing the gauge. There are two classes I'd consider core, as they don't do any additional processing. The Dropwizard Gauge has a [callback-based API](http://metrics.dropwizard.io/3.1.0/manual/core/#gauges). The Dropwizard Counter has [`inc` and `dec`](http://metrics.dropwizard.io/3.1.0/manual/core/#counters) methods, which actually makes it a gauge in Prometheus terms. The uses cases for other Dropwizard gauge types are handled differently in Prometheus, the Dropwizard Meter would be a Prometheus Counter combined with the processing done with the PromQL `rate` function for example.

There is one other difference with many other monitoring and instrumentation systems. As is true throughout Prometheus, full 64bit floating point values are supported.

PromQL offers a number of functions and operators to deal with gauges. In fact except for the handful of functions that are for use solely with counters (such as `rate`), all functions can be sanely used with a gauge. Functions of note include `deriv` to calculate the value's rate of change using simple linear regression, and `predict_linear` which takes that a step further and projects into the future. Aggregation across series is possible with operators like `max`, `min` and `avg` and aggregation across time is possible with functions such as `max_over_time`, `min_over_time` and `avg_over_time`.
