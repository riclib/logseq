---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/get-thee-to-a-nannary
author: [[Brian Brazil]] 
---
> NaN is just a number in Prometheus.

# Get thee to a NaNnary


NaN is just a number in Prometheus.

Some monitoring systems use NaN as a null or missing value, however in Prometheus NaN is just another floating point value. The way Prometheus represents missing data is to have the data, uhm, missing. Prometheus supports all 64-bit floating point values, including positive infinity, negative infinity, and NaN.

There's two related cases where you're likely to run across NaN. The first is when getting average latency from the sum and count or a summary and histogram with `rate(my_sum[5m])/rate(my_count[5m])`. If there have been no recent events then rate of the count is 0 and you're dividing by zero, and the result is thus correctly NaN. The second case is for the quantiles for summary metrics, which will also be NaN if there have been no recent events.

A question that comes up every now and then is how to filter out NaNs, as any math involving a NaN will return NaN. Per standard floating point semantics you could take advantage of the NaN's unique property that NaN != NaN. However the use cases for this generally turn out to be averaging of either averages or quantiles, neither of which are statistically valid.

Accordingly don't try to do

avg by (job)(
    rate(my\_sum\[5m\])
  / 
    rate(my\_count\[5m\])
)

as it is meaningless, instead do

  sum by (job)(rate(my\_sum\[5m\]))
/
  sum by (job)(rate(my\_count\[5m\]))

that is sum and then divide. In general, always do division last.

Relatedly if a NaN manages to get into the input of a function or operator that does math on values the result will be NaN. In such a case eliminate the source of the NaN, rather than trying to workaround the bad data further downstream.

There are places where's special handling for NaN values in PromQL, so that the behaviour is as expected. `min` and `max` will consider a NaN value to be bigger/smaller than all other numbers, respectively. `sort` and `sort_desc` are not actually symmetrical, NaNs are always sorted to the bottom. Similarly `bottomk` and `topk` will consider a NaN values to be bigger/smaller than all other numbers, respectively. Put another way, as long as you have at least `k` non-NaN values, `bottomk` and `topk` will not return a NaN. At one point `changes` also needed bug fixing to handle `NaN` correctly.

There is one place where a NaNs can have special meaning to Prometheus, and that's the markers used as part of staleness handling. However this is an implementation detail. That the particular bit pattern we use in our staleness implementation happens to be a NaN is never visible to users of PromQL, though remote storage implementations may have to care about this if they're doing any math themselves.

_Need help eliminating a NaN result? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
