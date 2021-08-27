---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/composing-range-vector-functions-in-promql
author: [[Brian Brazil]] 
---
# Composing range vector functions in PromQL

> If you try and do max_over_time(rate(my_counter_total[5m])[1h]) or predict_linear(rate(my_counter_total[5m])[1d], 3600) in Prometheus it won't work. How can you combine these functions?

---
If you try and do `max_over_time(rate(my_counter_total[5m])[1h])` or `predict_linear(rate(my_counter_total[5m])[1d], 3600)` in [Prometheus](https://www.prometheus.io/) it won't work. How can you combine these functions?

There are two general types of [functions](https://prometheus.io/docs/querying/functions/) in PromQL that take timeseries as input, those that take a vector and return a vector (e.g. `abs`, `ceil`, `hour`, `label_replace`), and those that take a range vector and return a vector (e.g. `rate`, `deriv`, `predict_linear`, `*_over_time`).

There are no functions that take a range vector and return a range vector, nor is there a way to do any form of subquery ([prior to Prometheus 2.7.0](https://www.robustperception.io/how-much-of-the-time-is-my-network-usage-over-a-certain-amount)). Even with support for subqueries, you wouldn't want to use them regularly as they'd be expensive. So what to do instead?

The answer is to use a [recording rule](https://prometheus.io/docs/querying/rules/) for the inner function, and then you can use the outer function on the time series it creates.

You can add the path of a file to read rules from to your `prometheus.yml`:
```yaml
rule_files:
  - my_rules.rules
```
And put your rules in that file:

instance:my_counter:rate5m = rate(my_counter_total{job="myjob"}[5m])

# You can use this new metric now in rules, alerts or graphs.
```yaml
instance:my_counter:predict_linear1d_1h_rate5m = 
    predict_linear(instance:my_counter:rate5m{job="myjob"})[1d], 3600)
```

One thing to keep in mind is the time ranges that the inner function operates on. A rate over ten minutes is going to be much smoother than a rate over one minute.

Finally even though you now know this technique don't forget, to always [rate then sum](https://www.robustperception.io/rate-then-sum-never-sum-then-rate/)!
