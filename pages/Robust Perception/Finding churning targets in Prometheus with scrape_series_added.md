---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/finding-churning-targets-in-prometheus-with-scrape_series_added
author: [[Brian Brazil]] 
---
> Prometheus 2.10 has a new metric to make finding churn easier.

# Finding churning targets in Prometheus with scrape_series_added


Prometheus 2.10 has a new metric to make finding churn easier.

The new `scrape_series_added` metric indicates how many new series were created in a given scrape. Due to various technicalities it may under or over-report, however for "normal" usage it should be quite useful for discovering misbehaving targets - without having to wait for a block to be compacted so you can use the more accurate [tsdb analyze](https://www.robustperception.io/using-tsdb-analyze-to-investigate-churn-and-cardinality).

So how does it work? Every time a series is added to the scrape cache for a target, it counts as a new series. So if a brand new target appears all its series will be counted on the first successful scrape, catching lots of targets being created and destroyed. If a target has series added and removed across scrapes every new series that wasn't in the previous successful scrape will be counted, catching churn in the series returned.  Looking at `scrape_samples_scraped` alone wouldn't show you that.

To use it you can employ an expression like:

`topk(10, sum without(instance)(sum_over_time(scrape_series_added[1h])))`

This will give a per-target churn value over the past hour, and aggregate it up ignoring the instance label and then find the biggest 10 churners.

Note that is is possible for certain types of failed scrapes to create series, even if the associated samples are never ingested, `up` and `scrape_*` are not included in this metric, and that restarting Prometheus or changing the scrape configuration can clear the scrape cache. Future work may improve the accuracy of this metric, but it's still quite useful as-is.

_Need help optimising your Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
