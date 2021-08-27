---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/rate-then-sum-never-sum-then-rate
author: [[Brian Brazil]] 
---
# Rate then sum, never sum then rate

> There's a common misunderstanding when dealing with Prometheus counters, and that is how to apply aggregation and other operations when using the rate and other counter-only functions.

---
There's a common misunderstanding when dealing with [Prometheus](https://prometheus.io/) counters, and that is how to apply aggregation and other operations when using the `rate` and other counter-only functions.

Aggregation is core functionality of Prometheus, and it's most commonly applied to counters. As you'll recall from a [previous article](http://www.robustperception.io/how-does-a-prometheus-counter-work/) counters only go up and reset. This has implications for what order you apply operations in.

Let's say you are aggregating up the rate of requests across all of your Node exporters. The individual rates would be:
```
rate(http_requests_total{job="node"}[5m])
```
Now to aggregate those, you'd do:
```
sum by (job)(rate(http_requests_total{job="node"}[5m]))  # This is okay
```
Seems simple, right?

## How not to do it

A common mistake is to try to take the sum and then the rate:

rate(sum by (job)(http_requests_total{job="node"})[5m])  # Don't do this

Even if you've worked around this being invalid expression with a recording rule, the real problem is what happens when one of the servers restarts. The counters from the restarted server will reset to 0, the sum will decrease, which will then be treated by `rate` as a counter reset and you'd get a large spurious spike in the result.

For the same reason:
```
rate(counter_a[5m] + counter_b[5m])        # Don't do this
```
is incorrect, instead do:
```
rate(counter_a[5m]) + rate(counter_b[5m])  # This is okay
```
Similar applies to all other functions, operators and aggregates such as `min`, `max`, `avg`, `ceil`, `histogram_quantile`, `predict_linear`, division etc.

## Rule of Thumb

To help keep you on the straight and narrow, remember this: The only mathematical operations you can safely directly apply to a counter's values are `rate`, `irate`, `increase`, and `resets`. Anything else will cause you problems.
