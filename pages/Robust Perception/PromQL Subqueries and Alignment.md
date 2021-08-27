---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/promql-subqueries-and-alignment
author: [[Brian Brazil]] 
---
> Subqueries were added to PromQL a while back, but there's more to this feature than it'd seem.

# PromQL Subqueries and Alignment


Subqueries were added to PromQL a while back, but there's more to this feature than it'd seem.

For the longest time subqueries had fundamental issues that made them challenging to add.

Firstly from a semantic standpoint what interval should they be executed on? The ultimate answer was to use the global evaluation interval as a default, and allow the user to override if they want. Not too hard.

The big problem though was performance. Historically for a range query it'd calculate each step completely independently, so you'd end up recalculating the same subquery data again and again for each step. So if say you had a range query of `avg_over_time((a_gauge == bool 2)[1h:10s])` with a step of 10s over an hour, you'd end up calculating 3600/10=360 samples for the subquery evaluation within each step then multiplied by the 360 steps for 129,600 samples in all. Without the duplication we'd only need to calculate 720 samples or two hours worth of 10s step subquery output.

A solution was in sight with [Prometheus 2.3.0](https://www.robustperception.io/new-features-in-prometheus-2-3-0), where I changed evaluation of range queries to be AST node by AST node (that is completely evaluate everything for one function/operator for all steps, before evaluating the next one) which brought substantial performance gains to PromQL in general.

So we're done then, right? Performance would no longer be a showstopper as we can calculate all the subquery data at once, and subqueries could be added as a feature?

Not quite. The above example has the same value for the range query's step and the subquery's step. If the subquery's step doesn't divide evenly into the query range's this doesn't work, and this will often be the case. Say that the range query was starting at t=100, with a 3s step. The subquery was going back 20s, with a 5s step. So the first range query step needs samples at t=80, 85, 90, 95 and 100. The second step needs samples at t=83, 88, 83, 98, and 103. This gets us back to situation similar to where we were when each range query step was calculated completely independently, losing the wins of the 2.3.0 improvements.

For more fun, there's also the possibility of nested subqueries.

And there's another problem. One of the design principles of PromQL and the Prometheus query APIs is that range queries are merely syntactic sugar over instant query. Sure that got inverted in the implementation of 2.3.0 for performance reasons, but the principle stands and there's even unittests to avoid accidentally violating it in future.

So we need something that's not only efficient, but maintains consistent results whether an evaluation at a given timestamp is as an instant query or just one step of a range query.

The solution I came up with was to completely ignore the alignment of the surrounding query. That is whether the query range step being evaluated for is t=100 or t=103 it'd be getting data from the same subquery output. So in the above example it'd always be (say) samples at t=80, 85, 90, 95, 100, 105 etc. and we provide an appropriate range vector as needed to the next stage in the PromQL evaluation - just like a normal time series.

This is conveniently also consistent with another design decision of Prometheus, which is you can configure your scrape and evaluation intervals but it's unspecified at exactly what phase Prometheus will actually perform things at. So t=80, 85 is just as valid as t=84.12, 89.12. This was done to allow us to spread scrape and evaluation load around, as trying to do all scrapes at exactly the same time was unwise (which I fixed all the way back in 0.8.0), but also helps here. As there's no processes that we might wish to spread the load of, subqueries currently align everything with t=0 as that's as good a choice as any. That's technically unspecified just as with phases elsewhere, but I don't currently see a reason that we'd want to change it.

So if you ever find yourself wondering why software X doesn't add feature Y, keep in mind that there's often more to it than it would appear at first.

_Want help making your PromQL more efficient? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
