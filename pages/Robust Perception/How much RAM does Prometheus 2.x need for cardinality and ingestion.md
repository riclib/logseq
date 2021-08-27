---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/how-much-ram-does-prometheus-2-x-need-for-cardinality-and-ingestion
author: [[Brian Brazil]] 
---
> I previously looked at ingestion memory for 1.x, how about 2.x?

# How much RAM does Prometheus 2.x need for cardinality and ingestion?


I previously looked at [ingestion memory for 1.x](https://www.robustperception.io/how-much-ram-does-my-prometheus-need-for-ingestion), how about 2.x?

Prometheus 2.x has a very different ingestion system to 1.x, with many performance improvements. This time I'm also going to take into account the cost of cardinality in the head block. To start with I took a [profile](https://www.robustperception.io/analysing-prometheus-memory-usage) of a Prometheus 2.9.2 ingesting from a single target with 100k unique time series:

[![](https://www.robustperception.io/wp-content/uploads/2019/04/viewport-429x640.png)](https://www.robustperception.io/wp-content/uploads/2019/04/viewport.png)

This gives a good starting point to find the relevant bits of code, but as my Prometheus has just started doesn't have quite everything. From here I can start digging through the code to understand what each bit of usage is.

So `PromParser.Metric` for example looks to be the length of the full timeseries name, while the `scrapeCache` is a constant cost of 145ish bytes per time series, and under `getOrCreateWithID` there's a mix of constants, usage per unique label value, usage per unique symbol, and per sample label. The usage under `fanoutAppender.commit` is from the initial writing of all the series to the WAL, which just hasn't been GCed yet. One thing missing is chunks, which work out as 192B for 128B of data which is a 50% overhead.

From here I take various worst case assumptions. For example half of the space in most lists is unused and chunks are practically empty. To simplify I ignore the number of label names, as there should never be many of those. This works out then as about 732B per series, another 32B per label pair, 120B per unique label value and on top of all that the time series name twice. Last, but not least, all of that must be doubled given how Go garbage collection works.

That's cardinality, for ingestion we can take the scrape interval, the number of time series, the 50% overhead, typical bytes per sample, and the doubling from GC. Given how head compaction works, we need to allow for up to 3 hours worth of data.

Rather than having to calculate all of this by hand, I've done up a calculator as a starting point:

This shows for example that a million series costs around 2GiB of RAM in terms of cardinality, plus with a 15s scrape interval and no churn around 2.5GiB for ingestion.

_Need help sizing your Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
