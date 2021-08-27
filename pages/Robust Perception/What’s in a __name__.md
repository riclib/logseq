---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/whats-in-a-__name__
author: [[Brian Brazil]] 
---
# What’s in a __name__?

> You may have noticed that most PromQL functions and operators remove the metric name in their result. Let's look at why.

---
You may have noticed that most PromQL functions and operators remove the metric name in their result. Let's look at why.

Let's take a well known metric, `process_cpu_seconds_total`, which is the amount of CPU time in seconds that a process has used since startup.

If I multiply that by 0 or take a `rate()`, the resultant metric is no longer the amount of CPU time in seconds that a process has used since startup - so `process_cpu_seconds_total` no longer makes sense as a metric name. PromQL doesn't know what name would make sense, so the metric name (contained in the `__name__` label) is removed.

There are cases such as multiplying by 1 or adding 0 where the time series remains the same, in these cases the metric name is still removed for consistency. This follows the general rule that you should always know what labels are in play, and different label semantics based on changes in sample values would violate that.

There are some classes of operators that preserve the metric name, as they can't change the metric - merely filter it. These are the binary comparison operators like `==` and `<`, the binary logical/set operators `and`, `unless`, and `or`, and the filtering aggregation operators `topk` and `bottomk`.

There are only three function that preserve the metric name: `label_replace`, `sort` and `sort_desc`. The sorting functions are cosmetic, there's no semantic difference from PromQL's standpoint to any particular ordering of time series within a vector so there's no reason to remove the metric name.  `label_replace` is for messing around with labels, so it's presumed you know what you're doing.

Finally I'd like to talk about the most common reason users run into this. Users are trying to perform math across multiple metrics using an expression like `rate({__name__=~"pool_.*_total"}[5m])`, where there's a regex on the metric name. This is an anti-pattern, as you're meant to know your metric names a-priori. The only time you should use a regex against metric names is when doing performance debugging or metric exploration.

The correct way to handle this is to use a label to distinguish the different pools (or whatever it is you're trying to distinguish), rather than encoding them inside the metric name. If it's not possible to do this in the code, you can workaround using `metric_relabel_configs` with `replace` actions to a) extract out the label and then b) set the metric name to a single value.
