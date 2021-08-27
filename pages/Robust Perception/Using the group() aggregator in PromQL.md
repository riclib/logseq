---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/using-the-group-aggregator-in-promql
author: [[Brian Brazil]] 
---
> Prometheus 2.20 added a group aggregator. What is it for?

# Using the group() aggregator in PromQL


[Prometheus 2.20](https://www.robustperception.io/new-features-in-prometheus-2-20-0) added a `group` aggregator. What is it for?

If you wanted to count the number of unique values a label has, such as say the number of values the `cpu` label had in `node_cpu_seconds_total` per `instance` the standard pattern is:

count without(cpu) (
  count without(mode) (node\_cpu\_seconds\_total)
)

That is first you aggregate away any other labels that don't matter with the inner aggregation (here just the `mode` label), and then count the resulting series with the outer aggregation.

The thing is though that the inner aggregation doesn't have to be `count`. It could be `sum`, `max`, or even `stddev`, because it's not the numeric result of the aggregation that we care about but instead the labels on the resultant time series. This is where `group` comes in. `group` always produces 1 as a the value of the aggregation group, but more importantly signifies for anyone reading the PromQL expression that it is the aggregation's label handling that is what's important rather than the numeric result:

count without(cpu) (
  group without(mode) (node\_cpu\_seconds\_total)
)

Finally, as the value is 1 you could also use `sum` for the outer aggregation similarly to how you could for [info metrics](https://www.robustperception.io/why-info-style-metrics-have-a-value-of-1).

_Have PromQL questions?_ _[Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
