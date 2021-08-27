---
created: [[Aug 5th, 2021]]
type: #clipping
status: reviewed
tags: prometheus 
source: https://www.robustperception.io/limiting-promql-resource-usage
author: [[Brian Brazil]] 
---
> Prometheus has gained a number of features to limit the impact of expensive PromQL queries.

# Limiting PromQL resource usage

Prometheus has gained a number of features to limit the impact of expensive PromQL queries.

If someone runs a resource intensive query, such as aggregating across thousands of individual time series over a long time period, it's not unknown for it to eat CPU and RAM. In the worst case, Prometheus can get killed by the kernel OOMkiller.

In the beginning Prometheus had no features to limit PromQL queries. The first was [added by myself in 2014](https://github.com/prometheus/prometheus/commit/f114bbd4e70a6b0dbfd6bff4cc3c4561e03cf912), a limit on the number of steps that a query\_range call could have to 11,000 as due to a bug I had managed to perform a query with a 1s step from 1970 to now - which unsurprisingly took out my Prometheus. Thus I added a sanity check, as no one has a monitor with that much horizontal resolution and it should also be plenty for most batch needs.

The second limit was a [time limit added in 2015](https://github.com/prometheus/prometheus/commit/fa1e90003b9e05c697b4fb425f276968d06ff5b8), which is user configurable. While the implementation has changed to use Go contexts and there have been other improvements in where it's checked, the core of this has remained the same.

Later in 2015 a user configurable limit was added on the [number of concurrent queries](https://github.com/prometheus/prometheus/commit/9ab1f6c690940187b227cdd6ec591349abf0f5aa) that could be executed, delaying queries if needed. This avoids queries fighting over the same resources, slowing everything down.

Not much happened from then until recently. There were various performance improvements from a new TSDB to my own work on optimising PromQL earlier this year. The more efficient PromQL implementation actually caused a few out of memory issues, as previously some large slow queries were hitting the timeout but were now quickly eating up memory!

The most recent addition was in Prometheus 2.5.0, a limit on [how many samples a PromQL query can have in memory at once](https://github.com/prometheus/prometheus/pull/4513). As samples are generally responsible for most of the memory usage of a query, this is a simple yet powerful way to halt abusive queries. Other memory users include timeseries labels, and various smaller things like the slices to hold the time series, iterators, and caches.

As each raw sample is 16 bytes in memory during evaluation, you can use this and the concurrency limit to get an idea of how much RAM you might need to allow for queries. The default concurrency is 20, and sample limit is 50M which would be around 16GB. However given how Go's garbage collection works you need to double that, and in the worst case the difference between length and capacity of the slices containing the samples may cause another doubling. So you could be talking 32GB for samples in the worst case - but that's presuming 20 heavy queries running simultaneously in the worst possible way. In reality most queries will be much smaller, and you can always drop the concurrency limit to nearer your number of CPU cores. It's best to think of this more as a way to halt individual queries that would otherwise eat up 10 GB (or more) of RAM, rather than try to microoptimise and try to distinguish between 100MB and 200MB of query memory usage.