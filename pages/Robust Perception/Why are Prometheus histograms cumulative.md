---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/why-are-prometheus-histograms-cumulative
author: [[Brian Brazil]] 
---
> Have you ever wondered why the buckets in histograms are not just counters of events that fall into each bucket?

# Why are Prometheus histograms cumulative?


Have you ever wondered why the buckets in histograms are not just counters of events that fall into each bucket?

Let us say that you have a histogram with the default buckets recording latency, and had observed events of size 0.04s, 0.2s, 0.3s, 1s and 5s. The text format output would look like:

\# HELP example\_latency\_seconds Some help text
# TYPE example\_latency\_seconds histogram
example\_latency\_seconds\_bucket{le="0.005"} 0.0
example\_latency\_seconds\_bucket{le="0.01"} 0.0
example\_latency\_seconds\_bucket{le="0.025"} 0.0
example\_latency\_seconds\_bucket{le="0.05"} 1.0
example\_latency\_seconds\_bucket{le="0.075"} 1.0
example\_latency\_seconds\_bucket{le="0.1"} 1.0
example\_latency\_seconds\_bucket{le="0.25"} 2.0
example\_latency\_seconds\_bucket{le="0.5"} 3.0
example\_latency\_seconds\_bucket{le="0.75"} 3.0
example\_latency\_seconds\_bucket{le="1.0"} 4.0
example\_latency\_seconds\_bucket{le="2.5"} 4.0
example\_latency\_seconds\_bucket{le="5.0"} 5.0
example\_latency\_seconds\_bucket{le="7.5"} 5.0
example\_latency\_seconds\_bucket{le="10.0"} 5.0
example\_latency\_seconds\_bucket{le="+Inf"} 5.0
example\_latency\_seconds\_count 5.0
example\_latency\_seconds\_sum 6.54

The 0.05 bucket has one sample, as you'd expect. But the 0.25 bucket has two samples, even though there's only one sample in the range 0.1 < x <= 0.25. This is as the `le` stands for less than or equal to, and each bucket includes the samples of the buckets below it. Thus the +Inf bucket is the total number of samples, which will match up with the `_count`.

This is surprising to some, as they expect histograms to be non-cumulative. So why does the format use cumulative histograms, rather than the more intuitive non-cumulative alternative?

The answer is operational. Cardinality is always something to consider with labels, and a histogram by default will have a cardinality of 10 for its buckets. If additional labels are added to the histogram, or more buckets are added, then histograms can get rather expensive. Having cumulative histograms means that some buckets can be dropped at ingestion time, reducing the cost to Prometheus while still allowing (somewhat less accurate) quantiles to be calculated. This buys you time to get the application code changed to reduce the cardinality.

For example if we wanted to drop all the buckets below 100ms at ingestion time, the following relabelling configuration could be used:

scrape\_configs:
 - job\_name: 'my\_job'
   static\_configs:
     - targets:
       - my\_target:1234
   metric\_relabel\_configs:
   - source\_labels: \[ \_\_name\_\_, le \]
     regex: 'example\_latency\_seconds\_bucket;(0\\.0.\*)'
     action: drop

You can drop as many or as few buckets as you like, however the +Inf bucket is required for `histogram_quantile`.

The potential high cardinality of histograms is also one reason a histogram has a `_sum` and `_count`. Even if you drop all the bucket time series, you can still calculate an average latency with just those two time series.

In addition this approach makes it easy to calculate the proportion of events below a given bucket, for example the ratio of events below 1s could be calculated with

    example\_latency\_seconds\_bucket{le="1.0"}
/ ignoring (le)
    example\_latency\_seconds\_bucket{le="+Inf"}

_Want help planning for cardinality? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
