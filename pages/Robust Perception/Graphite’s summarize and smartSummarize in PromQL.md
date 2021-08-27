---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/graphites-summarize-and-smartsummarize-in-promql
author: [[Brian Brazil]] 
---
> How do you convert summarizeinto PromQL?

# Graphite’s summarize and smartSummarize in PromQL


How do you convert `summarize`into PromQL?

Graphite's [`summarize`](https://graphite.readthedocs.io/en/1.1.5/functions.html#graphite.render.functions.summarize) function buckets data for individual series based on time. The equivalent functionality in PromQL is provided by the [various `_over_time` functions](https://prometheus.io/docs/prometheus/2.15/querying/functions/#aggregation_over_time). In addition when converting we need to allow for the different the execution models of PromQL and Graphite. Graphite works on producing a whole graph at once. PromQL executes many queries independently for each vertical time slice, which you could then graph.

So taking an example from the Graphite docs

render?target=summarize(counter.errors, "1hour") # total errors per hour

would translate to

query\_range?query=sum\_over\_time(counter\_errors\[1h\])&step=1h

on the Prometheus API. Notice that the `step` is an explicit thing you can choose with the Prometheus `query_range` API, and can differ from what's inside the query.

These two API examples are not a fully fair comparison though. The Prometheus `query_range` API requires that a start time and end time be specified whereas Graphite defaults to the last 24 hours, so

query\_range?query=sum\_over\_time(counter\_errors\[1h\])&step=1h
  &start=2020-01-01T00:00:00Z&end=2020-01-02T00:00:00Z

would match up with

render?target=summarize(counter.errors, "1hour")
  &from=midnight+20200101&until=midnight+20200102

The dates make things verbose, so it's a good thing that API URLs are usually procedurally generated.

There's also a difference in instrumentation design apparent in this example, in which the errors are coming from a client-side calculated gauge. By contrast Prometheus would use a counter for such a use case, and the query

query\_range?query=increase(my\_errors\_total\[1h\])&step=1h

to calculate the number of errors on the Prometheus side. In Graphite you would use [nonNegativeDerivative](https://graphite.readthedocs.io/en/1.1.5/functions.html#graphite.render.functions.nonNegativeDerivative) for this:

render&target=summarize(nonNegativeDerivative(my\_errors\_total), "1h")

`summarize` offers a number of functions via the `func` URL parameter, with the mapping to PromQL being reasonably obvious in most cases. We've already seen that `sum` is `sum_over_time` in PromQL.

`average` is `avg_over_time` , `median` is `quantile_over_time(0.5, ...)`,  `min` is `min_over_time`, `max` is `max_over_time`, `stddev` is `stddev_over_time`, and `count` is `count_over_time`. `range` is a little more involved as `max_over_time() - min_over_time()`.

There's no equivalent for `last`, as having the last value of a series at an unknown timestamp isn't particularly useful for doing calculations on. PromQL staleness is how that class of problem is approached in Prometheus. I can't come up with plausible use cases for `multiply` or `diff` (which [takes first value, and subtracts all the subsequent values from it](https://github.com/graphite-project/graphite-web/blob/52c0fd11a18de633e28e389d394ceba45a8e2831/webapp/graphite/functions/safe.py#L14-L19)) or find examples uses of them online. I believe these are here as they're in the list of aggregation functions which are used elsewhere, not because they're useful.

The [`smartSummarize`](https://graphite.readthedocs.io/en/1.1.5/functions.html#graphite.render.functions.smartSummarize) function has all the same logic as `summarize`, the only difference is that the alignment parameters have been reworked. So given that Prometheus has different parameters to Graphite anyway, it's all the same in this particular context.

So for common use cases, converting between Graphite and PromQL isn't particularly difficult.

_Have a question about PromQL? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
