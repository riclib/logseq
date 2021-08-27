---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/common-query-patterns-in-promql
author: [[Brian Brazil]] 
---
# Common query patterns in PromQL

> For day to day use, there's only a handful of PromQL patterns you need to know. Let's look at them.

---
For day to day use, there's only a handful of PromQL patterns you need to know. Let's look at them.

In these examples I'll presume you have simple unlabelled metrics coming from multiple instances, that you wish to aggregate up to the job level. If there's additional labels you want to aggregate away, add them to the `without` clause.

## Gauges

Usually you'll want to aggregate up the sum or average of a gauge:
```
sum without (instance)(my_gauge)
avg without (instance)(my_gauge)
```

Similarly for `min` and `max`.

## Counters

For counters you need to take a rate, and then aggregate:
```
sum without (instance)(rate(my_counter_total[5m]))
```
## Summary

A summary contains `_sum` and `_count` counters, from which we can calculate the average event size. This is often the average latency:
```
  sum without (instance)(rate(my_summary_latency_seconds_sum[5m]))
/
  sum without (instance)(rate(my_summary_latency_seconds_count[5m]))
```
This sums up the rates of the two constituent counters, and then divides them.

This query pattern works in other situations too, such as a failure ratio:
```
  sum without (instance)(rate(my_events_failed_total[5m]))
/
  sum without (instance)(rate(my_events_total[5m]))
```

## Histogram

A histogram contains buckets, from which we can calculate say the 90th percentile latency:
```
histogram_quantile(
    0.9, 
    sum without (instance)(rate(my_histogram_latency_seconds_bucket[5m])))
```
We take the rate of the bucket counters,  aggregate up and then calculate the quantile.

A histogram also contains a `_sum` and `_count`, so the summary query from above will work here too.

So there we are, just 4 common patterns for you to remember that should cover >80% of your needs!
