---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/how-does-a-prometheus-histogram-work
author: [[Brian Brazil]] 
---
> We looked previously at the counter, gauge, and summary, how does the Prometheus histogram work?

# How does a Prometheus Histogram work?


We looked previously at the [counter](https://www.robustperception.io/how-does-a-prometheus-counter-work/), [gauge](https://www.robustperception.io/how-does-a-prometheus-gauge-work), and [summary](https://www.robustperception.io/how-does-a-prometheus-summary-work), how does the Prometheus histogram work?

The histogram has several similarities to the summary. A histogram is a combination of various counters. Like summary metrics, histogram metrics are used to track the size of events, usually how long they take, via their `observe` method. There's usually also the exact utilities to make it easy to time things as there are for summarys. Where they differ is their handling of quantiles.

Here's an example of the exposition format from Prometheus itself, which also happens to have a `handler` label:

\# HELP prometheus\_http\_request\_duration\_seconds Histogram of latencies for HTTP requests.
# TYPE prometheus\_http\_request\_duration\_seconds histogram
prometheus\_http\_request\_duration\_seconds\_bucket{handler="/",le="0.1"} 25547
prometheus\_http\_request\_duration\_seconds\_bucket{handler="/",le="0.2"} 26688
prometheus\_http\_request\_duration\_seconds\_bucket{handler="/",le="0.4"} 27760
prometheus\_http\_request\_duration\_seconds\_bucket{handler="/",le="1"} 28641
prometheus\_http\_request\_duration\_seconds\_bucket{handler="/",le="3"} 28782
prometheus\_http\_request\_duration\_seconds\_bucket{handler="/",le="8"} 28844
prometheus\_http\_request\_duration\_seconds\_bucket{handler="/",le="20"} 28855
prometheus\_http\_request\_duration\_seconds\_bucket{handler="/",le="60"} 28860
prometheus\_http\_request\_duration\_seconds\_bucket{handler="/",le="120"} 28860
prometheus\_http\_request\_duration\_seconds\_bucket{handler="/",le="+Inf"} 28860
prometheus\_http\_request\_duration\_seconds\_sum{handler="/"} 1863.80491025699
prometheus\_http\_request\_duration\_seconds\_count{handler="/"} 28860

The `_sum` and `_count` work in exactly the same way as for a summary, and they can be used to produce an average duration over the past five minutes:

  rate(prometheus\_http\_request\_duration\_seconds\_sum\[5m\])
/
  rate(prometheus\_http\_request\_duration\_seconds\_count\[5m\])

There are very rare cases where the `_sum` won't be present, such as in certain metrics from the MySQLd exporter.

The interesting part of the histogram are the `_bucket` time series, which are the actual histogram part of the histogram. More particularly they're counters which form a cumulative histogram, `le` stands for less than or equal to. So 26688 requests took less than or equal to 200ms, 27760 requests took less than or equal to 400ms, and there were 28860 requests in total. The values in the buckets will be monotonically non-decreasing with the `+Inf` bucket having the biggest value. The `+Inf` bucket must always be present, and will match the value of the `_count`.

To calculate say the 0.9 quantile (the 90th percentile) you would use:

histogram\_quantile(0.9, 
  rate(prometheus\_http\_request\_duration\_seconds\_bucket\[5m\])
)

One big advantage of histograms over summarys is that you can aggregate the buckets before calculating the quantile - taking care not to lose the `le` label:

histogram\_quantile(0.9, 
  sum without (handler)(
    rate(prometheus\_http\_request\_duration\_seconds\_bucket\[5m\])
  )
)

In addition to being aggregatable, histograms are cheaper on the client too as counters are fast to increment. So why not always use histograms? There's a [long answer](https://prometheus.io/docs/practices/histograms/#quantiles), but the short version is that with histograms you have to pre-choose your buckets, and the costs moves from the client to Prometheus itself due to bucket cardinality. The default ten buckets cover a typical web service with latency in the millisecond to second range, and on occasion you will want to adjust them. Here for example they have been overridden to better help track requests for PromQL, which have a two minute default timeout. Having more than ten buckets will give more accurate results, however it can also add up to a lot of time series. Particularly when combined with other labels.

With a real time monitoring system like Prometheus the aim should be to provide a value that's good enough to make engineering decisions based off. Knowing for example that the 90th percentile latency increased by 50ms is more important than knowing if the value is now 562ms or 563ms when you're oncall, and ten buckets is typically sufficient for this. If you need a perfect answer you can always calculate it from your logging system later on. In the event there's excessive buckets they can be dropped at ingestion, [as previously looked at](https://www.robustperception.io/why-are-prometheus-histograms-cumulative). In the more extreme cases you might ignore the `_bucket` series entirely, and rely on the average from `_sum` and `_count` instead.

In conclusion histograms allow for aggregatable calculation of quantiles, though you need to be a little wary of cardinality. Remember that a summary without quantiles is a cheap option if you don't really need a histogram.

_Unsure which metric type you should be using? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
