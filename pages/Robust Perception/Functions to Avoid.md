---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/functions-to-avoid
author: [[Brian Brazil]] 
---
# Functions to Avoid

> As PromQL has evolved, there are some functions that should no longer be used.

---
As PromQL has evolved, there are some functions that should no longer be used.

_Note: As of Prometheus 2.0 `count_scalar`, `drop_common_labels`, and `keep_common` have been removed._

The first of these is `count_scalar()`, which returns the number of series in an instant vector. Unlike the `count` aggregator, this returns a 0 if the vector is empty. If you are trying to alert on missing time series, `absent()` is a better way to do it.

Next up is `delta()` which calculates the difference between the first and last samples in a time range for a gauge, with some extrapolation. As it only uses two samples it's highly susceptible to outliers, which is not desirable. If you want to know how fast a gauge is changing over time, `deriv()` is a better choice as it uses a least squares regression. If you want to compare a gauge to a previous time the `offset` modifier allows for that, though you'll still be susceptible to outliers.

Finally there is `drop_common_labels()`. A key part of working with labelled time series is knowing which labels apply to what you're currently doing. `drop_common_labels()` and its counterpart the `keep_common` aggregation modifier will produce different labels depending on what the input is, which is thus undesirable. If there's labels you don't want showing up, adjust your `by`/`without` clause accordingly.
