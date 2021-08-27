---
created: [[Aug 5th, 2021]]
type: #clipping
status: #reviewed
tags: prometheus 
source: https://www.robustperception.io/alerting-on-gauges-in-prometheus-2-0
author: [[Conor Broderick]] 
---
> One of the major changes introduced in Prometheus 2.0 was that of staleness handling. Previously for instant vectors, Prometheus would return a point up to 5 minutes in the past which caused a number of different issues.

# Alerting on gauges in Prometheus 2.0
One of the major changes introduced in Prometheus 2.0 was that of staleness handling. ==Previously for instant vectors, Prometheus would return a point up to 5 minutes in the past which caused a number of different issues==.

==Certain metrics like `up` would persist for 5 minutes after the target had gone away==, resulting in false positives in alerts and stale data from downed targets being wrongly used in PromQL queries.

Staleness handling in Prometheus 2.0 resolves these issues by allowing for the tracking of monitoring targets and time series that have disappeared. This allows for a reduction in querying artifacts and an increase in alerting responsiveness.

However staleness handling also introduces new possibilities for false positives and false negatives to arise when alerting rules are poorly defined.

For instance, ==how do we write alerting rules to accurately detect if an instance is down==? If a scrape fails and the time series is marked as stale only for it to come back in the next scrape interval, this would result in an alert needlessly firing. This may also result in false positives, as a target rapidly appearing and disappearing will reset the `for` clause of the alert each time resulting in the alert not firing when it should be.

In order to avoid this issue we can make use of the query function [avg_over_time](https://prometheus.io/docs/prometheus/latest/querying/functions/#aggregation-_over_time) which we covered in [last week's blogpost](https://www.robustperception.io/what-percentage-of-time-is-my-service-down-for/) to discover for what percentage of time a given service was up or down for.

==Using `avg_over_time` in our alerting rules allows us to negate the issues described above by tracking how a particular time series appears over time rather than just the last known value of that time series==. This way gaps in our data resulting from failed scrapes and flaky time series will no longer result in false positives or negatives in our alerting.

For example, rather than having the alerting rule:
``` python
up{job="jobname"} < 1
```

which would trigger an alert as soon as one scrape failed, we could have:

``` python
avg_over_time(up{job="jobname"}\[5m\]) < 0.9
```
which would trigger an alert if the average of `up` was below 0.9 for the last 5 minutes of scrapes.