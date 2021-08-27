---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/are-increasing-timestamps-counters-or-gauges
author: [[Brian Brazil]] 
---
# Are increasing timestamps Counters or Gauges?

> Every now and then someone asks what metric type a increasing timestamp should be. Let's take a look.

---
Every now and then someone asks what metric type a increasing timestamp should be. Let's take a look.

As a brief reminder, in Prometheus counters are things that for the life of a process only ever go up. By contrast, gauges can go up and down.

So lets say you had a timestamp that only ever goes up, such as the last time the process handled a user query. Sounds kinda like a counter doesn't it?

However there are two other aspects to a counter to remember. The first is that counters start from 0. Secondly is that you should not care about the absolute value of a counter, but rather how fast it is increasing.

In the case of the timestamp for the last time we handled something, neither will apply. Unless you started on January 1st 1970 at midnight, it won't start at zero. Usually you have such a metric in order to compare it with the current time in order to detect that process is stuck, which involves the absolute value.

Another thing to consider is that timestamps can in fact go backwards, due to oddities like [leap seconds and NTP adjusting time on a machine](http://infiniteundo.com/post/25326999628/falsehoods-programmers-believe-about-time).

So timestamps should always be gauges, and generally reported using Unix timestamps in seconds.

Timestamps are not the only thing where this sort of logic applies, so always think if you care about how fast a time series is increasing or the absolute value. If you care about both, it's a gauge and you can use `deriv` to calculate how fast it is changing!
