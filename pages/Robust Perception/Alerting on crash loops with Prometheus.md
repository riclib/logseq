---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/alerting-on-crash-loops-with-prometheus
author: [[Brian Brazil]] 
---
> If your applications are restarting regularly, whether due to segfaults or OOMs, it'd be nice to know.

# Alerting on crash loops with Prometheus


If your applications are restarting regularly, whether due to segfaults or OOMs, it'd be nice to know.

Prometheus client libraries when running on Linux will usually expose a metric by the name of `process_start_time_seconds` which is the Unix time at which the process started. Every time it changes for a given target means that the process has restarted. Conveniently PromQL has a `changes()` function which can count the number of changes in a time series over time. You can then use this in alerts:

groups:
 - name: example
   rules:
    - alert: JobRestarting
     expr: avg without(instance)(changes(process\_start\_time\_seconds\[1h\])) > 3 
     for: 10m
     labels:
       severity: ticket

This calculates the restarts over an hour, and then tells you for each of your how jobs how many restarts each instance has on average. By alerting on the per-job average rather than on each instance individually this reduces the changes that a single bad machine would generate potentially spammy alert for issues that don't affect the job as a whole.

You could go a step further and count how many instances have restarted more than 3 times in the past hour, and alert if more than 10% of the instances in the job are in this state.

groups:
 - name: example
   rules:
    - alert: JobRestarting
     expr: avg without(instance)(changes(process\_start\_time\_seconds\[1h\]) > bool 3) > 0.1
     for: 10m
     labels:
       severity: ticket

This uses the `bool` modifier which returns 0 if a comparison fails or 1 if it succeeds, unlike normal comparisons which filter. These 0s and 1s can then be averaged without the instance label to give the proportion of each job's instances that have restarted often in the past hour.

There are two caveats with this approach. The first is that if the process is restarting faster than the scrape interval then some restarts may be missed, however any process restarting that frequently is going to trigger restart alerts anyway. Secondly this depends on the target labels not changing upon each restart, so if for example you had a new Kubernetes pod with new target labels on each restart then this technique would not work.

_Looking for advice on what to alert on? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
