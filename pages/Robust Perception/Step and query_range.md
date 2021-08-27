---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/step-and-query_range
author: [[Brian Brazil]] 
---
> Graphs from Prometheus use the query_range endpoint, and there's a non-trivial amount of confusion that it's more magic than it actually is.

# Step and query_range


Graphs from Prometheus use the query\_range endpoint, and there's a non-trivial amount of confusion that it's more magic than it actually is.

The [query range](https://prometheus.io/docs/prometheus/2.11/querying/api/#range-queries) endpoint isn't magic, in fact it is quite dumb. There's a query, a start time, an end time, and a step.

The provided query is evaluated at the start time, as if using the [query endpoint](https://prometheus.io/docs/prometheus/2.11/querying/api/#instant-queries). Then it's evaluated at the start time plus one step. Then the start time plus two steps, and so on stopping before the evaluation time would be after the end time. The results from all the evaluations are combined into time series, so if say samples for series A were present in the 1st and 3rd evaluations then both those samples would be returned in the same time series.

That's it. The query range endpoint is just syntactic sugar on top of the query endpoint\*. Functions like `rate` don't know whether they're being called as part of a range query, nor do they know what the step is. `topk` is across each step, [not the entire graph](https://www.robustperception.io/graph-top-n-time-series-in-grafana).

One consequence of this is that you must take a little care when choosing the range for functions like `rate` or `avg_over_time`, as if it's smaller then the step then you'll undersample and skip over some data. If using Grafana, you can use $\_\_interval to choose an appropriate value such as `rate(a_metric_total[$__interval])`.

\* This was the case until Prometheus 2.3.0, where I made significant performance improvements to PromQL. Query is now a special case of query range, however conceptually and semantically it's all still the same.  

_Want to take advantage of the Prometheus API? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
