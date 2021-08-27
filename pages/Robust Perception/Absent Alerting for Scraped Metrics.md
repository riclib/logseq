---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/absent-alerting-for-scraped-metrics
author: [[Brian Brazil]] 
---
> In the previous post we looked at dealing with when all the targets for a job had disappeared. What if you wanted to alert on specific metrics from one target disappearing?

# Absent Alerting for Scraped Metrics


In the [previous post](https://www.robustperception.io/absent-alerting-for-jobs/) we looked at dealing with when all the targets for a job had disappeared. What if you wanted to alert on specific metrics from one target disappearing?

It's best to [avoid metrics that appear and disappear](https://www.robustperception.io/existential-issues-with-metrics/), however it can happen that certain subsystems of a target don't always return all metrics that they should. It is possible to detect this situation by noticing that the `up` metric exists, but the metric in question does not. In addition you will want to check that `up` is 1, so that the alert doesn't spuriously fire when the target is down. If you already have down alerts for the job, there's no need to spam yourself with additional ones about missing metrics too.

The alert would look something like:

groups:
- name: example
  rules:
  - alert: MyJobMissingMyMetric
    expr: up{job="myjob"} == 1 unless my\_metric
    for: 10m

This uses `unless` which returns the left hand side, unless there's a matching metric on the right hand side.

As with other binary operators you can use `ignoring` if your metric has instrumentation labels that you wish to ignore, so for example `unless ignoring(method)`  would be appropriate if `my_metric` had a `method` label.

_Want advice on how to avoid missing metrics in the first place? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
