---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/combining-alert-conditions
author: [[Brian Brazil]] 
---
# Combining alert conditions

> Prometheus alerts use the same powerful PromQL expressions as queries and graphs. This can be used to produce sophisticated alerts.

---
Prometheus alerts use the same powerful [PromQL expressions](https://prometheus.io/docs/querying/basics/) as queries and graphs. This can be used to produce sophisticated alerts.

It's easy to produce alerts in Prometheus to cover cases like latency is too high. What if you wanted to also restrict the alert to a minimum amount of traffic or a time of day? The answer is the `and` [logical binary operator](https://prometheus.io/docs/querying/operators/#logical/set-binary-operators) combined with [vector matching](https://prometheus.io/docs/querying/operators/#vector-matching):
```yaml
groups:
- name: test.rules
  rules:
  - alert: LatencyTooHigh
    expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5 and ON(job) job:requests:rate5m{job="myjob"} > 100
```

The `and` returns the left hand side if there's a matching time series on the right hand side. The `ON(job)` means the job label must match on both sides, which it will here as there's the same job selector on each side.

Time restrictions are a little different, as there's no labels to match:

```yaml
groups:
- name: test.rules
  rules:
  - alert: LatencyTooHigh
    expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5 and ON() hour() > 9 < 17
```
Here `hour()` returns the hour of the day in UTC as a time series with no labels. We can use comparison operators to filter this down to the time we care about. Finally as there's no matching labels on either side of the `and`, we specify `ON()` to ignore all labels when matching.

As `and` doesn't care about how many matching time series there are on the right hand side, merely that there are either zero or not zero, this technique can be used in [ways we've looked at before](https://www.robustperception.io/alerting-on-down-instances/) to produce multiple alerts from just one alert definition.

Finally there's also the `unless` operator, which is has the opposite logic to `and`. It returns a time series on the left hand side if there's _not_ a matching time series on the right hand side.
w