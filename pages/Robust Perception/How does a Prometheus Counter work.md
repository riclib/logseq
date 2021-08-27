---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/how-does-a-prometheus-counter-work
author: [[Brian Brazil]] 
---
# How does a Prometheus Counter work?

> There are four standard types of metric in Prometheus instrumentation: Gauge, Counter, Summary and Histogram. Today we'll have a look at the principles around Counters, and how Prometheus differs from other monitoring systems.

---
There are [four](http://prometheus.io/docs/concepts/metric_types/) standard types of metric in Prometheus instrumentation: Gauge, Counter, Summary and Histogram. Today we'll have a look at the principles around Counters, and how Prometheus differs from other monitoring systems.

A counter counts things. Sounds simple, right?

That's not much use on its own though. What you really want to know is how fast it's counting, such as knowing that you're currently getting 12 requests per second. If at every request you increment the counter, how do you figure that out inside your monitoring system so that it can be graphed and alerted on?

There are three common approaches.

The first is that on a regular basis, such as once a minute, you extract the current value which goes to you monitoring system, and reset the counter to 0. This has a problem in that if the push fails, then you lose all information about that time period. This could leave you blind to a [micro burst](https://en.wikipedia.org/wiki/Micro-bursting_(networking)) of traffic. Additionally, if you've two systems pulling data from the counter for redundancy, each will only see about half the increments. That's not great.

The second approach is to use some form of [Moving Average](https://en.wikipedia.org/wiki/Moving_average#Weighted_moving_average), usually exponential. This means that recent data points have more importance than older data points. Depending on the phase and frequency of the increment pattern, relative to when the monitoring system samples information, you will get different results as not all data points are equal. This approach can handle multiple systems taking samples, but will lose information if a sample fails to be taken. This is better, but far from perfect.

Prometheus takes the third approach. A counter starts at 0, and is incremented. The client does no other calculations. At each scrape Prometheus takes a sample of this state. The `rate()` function in Prometheus looks at the history of time series over a time period, and calculates how fast it's increasing per second. This can handle multiple Prometheus servers taking samples, and if a scrape fails you'll lose resolution but not data as on the next successful scrape the increments haven't been lost or averaged away.

A common question is what happens when the process restarts and the counter is reset to 0? `rate()` will automatically handle this. Any time a counter appears to decrease it'll be treated as though there was a reset to 0 right after the first data point. This makes it important that it not be possible for Counters to be decremented, a Counter that has the potential to be decremented is in reality a Gauge. Other monitoring systems often throw away information when a counter reset happens, such as [Graphite's nonNegativeDerivative](http://graphite.readthedocs.org/en/latest/functions.html#graphite.render.functions.nonNegativeDerivative) function. `rate()` also has logic to handle samples not quite covering the entire time range you pass to `rate()`.

In addition to `rate()`, there's `increase()` and `irate()` which are the other two functions which operate on counters. While `rate()` returns per second values, `increase()` is a convenience function which returns the total across the time period. So for example `increase(my_counter_total[1h])` would return the increase in the counter per hour. `irate()` was [discussed previously](http://www.robustperception.io/irate-graphs-are-better-graphs/), it only looks at the last two samples and is useful for high precision graphs over short time frames. `resets()` tells you how often the counter resets, which can be useful for debugging.

Finally while we've only talked about counting requests here, Counters can be incremented by any non-negative floating point value. You could use them to track time spent, bytes processed, exceptions thrown or records walked.

I gave a more [in-depth talk](https://www.youtube.com/watch?v=67Ulrq6DxwA) on this topic at CloudNativeCon 2017 in Berlin.
