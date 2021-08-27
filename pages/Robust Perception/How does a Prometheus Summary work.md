---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/how-does-a-prometheus-summary-work
author: [[Brian Brazil]] 
---
> We looked previously at the counter and gauge, how does the Prometheus summary work?

# How does a Prometheus Summary work?


We looked previously at the [counter](https://www.robustperception.io/how-does-a-prometheus-counter-work/) and [gauge](https://www.robustperception.io/how-does-a-prometheus-gauge-work), how does the Prometheus summary work?

A summary is a combination of other types, to make common patterns simpler to use. A summary consists of two counters, and optionally some gauges. Summary metrics are used to track the size of events, usually how long they take, via their `observe` method. There's usually also utilities to make it easy to time things.

Let's look at an example of the exposition format:

\# HELP prometheus\_rule\_evaluation\_duration\_seconds The duration for a rule to execute.
# TYPE prometheus\_rule\_evaluation\_duration\_seconds summary
prometheus\_rule\_evaluation\_duration\_seconds{quantile="0.5"} 6.4853e-05
prometheus\_rule\_evaluation\_duration\_seconds{quantile="0.9"} 0.00010102
prometheus\_rule\_evaluation\_duration\_seconds{quantile="0.99"} 0.000177367
prometheus\_rule\_evaluation\_duration\_seconds\_sum 1.623860968846092e+06
prometheus\_rule\_evaluation\_duration\_seconds\_count 1.112293682e+09

The `_sum` and `_count` will (almost) always be there, these are both counters. `_count` is incremented by 1 on every `observe`, and `_sum` is incremented by the value of the observation. Here it looks like there have been about a billion observations since the process started, and they took 1.6M seconds in aggregate. This isn't much use on its own, but we can use `rate()` as we would with other counters. `rate(prometheus_rule_evaluation_duration_seconds_count[5m])` is the number of observations per second over the last five minutes on average, and `rate(prometheus_rule_evaluation_duration_seconds_sum[5m])` is how long they took per second on average. If we divide these we get the average duration of one observation:

  rate(prometheus\_rule\_evaluation\_duration\_seconds\_sum\[5m\]
/
  rate(prometheus\_rule\_evaluation\_duration\_seconds\_count\[5m\])

One thing to be aware of is that if there's no observations in a time period, then this would return NaN. This makes sense as we're dividing by 0. If you find this is getting in the way of other math keep in mind that you shouldn't average averages, and that instead you should do all the aggregation you need and then finally divide.

Overall summarys without quantiles are a nice cheap way to track latencies, amount of data transferred per request, records accessed etc. as it only uses two time series per labelset.

The samples with the `quantile` label are a bit more complicated, and will often not be present. These are quantiles of the observations, so the `{quantile="0.9"}` sample indicates that the 90th percentile is around 100us. What time period is this over though?

This is one of the big problems with calculating quantiles on the client side. If it's the quantile since the processes started, then the samples get less and less relevant to current conditions as time goes on. It's also not possible to do any math or aggregations once you have a quantile, so you can't stitch things back together given the history of the time series. How [Prometheus client libraries generally do it](https://github.com/prometheus/client_golang/blob/fa4aa9000d2863904891d193dea354d23f3d712a/prometheus/summary.go#L135) is to keep 10 quantile objects in memory. All observation are sent to all 10 objects, each tracking observations starting 1 minute from the next, and the oldest of these will contain up to 10 minutes worth of samples. Once the oldest is too old it is removed, and an empty quantile object started. The net effect of this is that quantiles returned by a Prometheus client library summary are over the observations in the last 10 minutes or so, with 1 minute of granularity. If there are no samples in a time period then NaN will be returned for the quantiles, as would be the same with dividing the `_sum` by the `_count` above.

This does ensure that recent observations aren't drowned out by older information, however like all client-side quantile systems it is relatively expensive in Go at about ~45x slower than two counters (21ns vs 930ns). It does have the advantage that you don't have to know the distribution of your observations in advance. Overall given the costs and that you can't aggregate such metrics further, I'd recommend sparing use of summaries with quantiles. That this feature exists is more historical than anything.

_Unsure which metric type you should be using? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
