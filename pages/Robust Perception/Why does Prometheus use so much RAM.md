---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/why-does-prometheus-use-so-much-ram
author: [[Brian Brazil]] 
---
> Users are sometimes surprised that Prometheus uses RAM, let's look at that.

# Why does Prometheus use so much RAM?


Users are sometimes surprised that Prometheus uses RAM, let's look at that.

More than once a user has expressed astonishment that their Prometheus is using more than a few hundred megabytes of RAM. A few hundred megabytes isn't a lot these days. Sure a small stateless service like say the node exporter shouldn't use much memory, but when you want to process large volumes of data efficiently you're going to need RAM. While Prometheus is a monitoring system, in both performance and operational terms it is a database.

The core performance challenge of a time series database is that writes come in in batches with a pile of different time series, whereas reads are for individual series across time. To make both reads and writes efficient, the writes for each individual series have to be gathered up and buffered in memory before writing them out in bulk. This has been [covered in previous posts](https://www.robustperception.io/how-much-ram-does-prometheus-2-x-need-for-cardinality-and-ingestion), however with new features and optimisation the numbers are always changing. As of Prometheus 2.20 a good rule of thumb should be around 3kB per series in the head. This allows not only for the various data structures the series itself appears in, but also for samples from a reasonable scrape interval, and remote write.

To put that in context a tiny Prometheus with only 10k series would use around 30MB for that, which isn't much. Indeed the general overheads of Prometheus itself will take more resources. On the other hand 10M series would be 30GB which is not a small amount.

That's just getting the data into Prometheus, to be useful you need to be able to use it via PromQL. This has also been [covered in previous posts](https://www.robustperception.io/limiting-promql-resource-usage), with the default limit of 20 concurrent queries using potentially 32GB of RAM just for samples if they all happened to be heavy queries. On top of that, the actual data accessed from disk should be kept in page cache for efficiency. For example if your recording rules and regularly used dashboards overall accessed a day of history for 1M series which were scraped every 10s, then conservatively presuming 2 bytes per sample to also allow for overheads that'd be around 17GB of page cache you should have available on top of what Prometheus itself needed for evaluation.

So you now have at least a rough idea of how much RAM a Prometheus is likely to need. Are there any settings you can adjust to reduce or limit this? The answer is no, Prometheus has been pretty heavily optimised by now and uses only as much RAM as it needs. If there was a way to reduce memory usage that made sense in performance terms we would, as we have many times in the past, make things work that way rather than gate it behind a setting. So there's no magic bullet to reduce Prometheus memory needs, the only real variable you have control over is the amount of page cache. However having to hit disk for a regular query due to not having enough page cache would be suboptimal for performance, so I'd advise against.

So how can you reduce the memory usage of Prometheus? Prometheus resource usage fundamentally depends on how much work you ask it to do, so ask Prometheus to do less work.

For example if you have high-cardinality metrics where you always just aggregate away one of the instrumentation labels in PromQL, remove the label on the target end. If you're ingesting metrics you don't need remove them from the target, or [drop them](https://www.robustperception.io/dropping-metrics-at-scrape-time-with-prometheus) on the Prometheus end. If you're scraping more frequently than you need to, do it less often (but not less often than once per 2 minutes). If you have recording rules or dashboards over long ranges and high cardinalities, look to aggregate the relevant metrics over shorter time ranges with recording rules, and then use `*_over_time` for when you want it over a longer time range - which will also has the advantage of making things faster.

_Have Prometheus performance questions? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
