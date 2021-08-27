---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/how-much-of-the-time-is-my-network-usage-over-a-certain-amount
author: [[Brian Brazil]] 
---
> The new subquery feature in Prometheus 2.7 makes this possible in one query.

# How much of the time is my network usage over a certain amount?


The new subquery feature in Prometheus 2.7 makes this possible in one query.

Let's say you wanted to know how much of the time in the last hour you were receiving over 1Mb/s, based on the average usage in the last five minutes. You can get the average usage over the last 5 minutes in bits/second with:

rate(node\_network\_receive\_bytes\_total\[5m\]) \* 8

You can change this into a 0 or 1 depending on whether it's above or below a value using the `bool` modifier:

rate(node\_network\_receive\_bytes\_total\[5m\]) \* 8 > bool 1000000

[Previously you would have had to create a recording rule](https://www.robustperception.io/composing-range-vector-functions-in-promql) to go to the next step, however subqueries allow you to do it on the fly:

(rate(node\_network\_receive\_bytes\_total\[5m\]) \* 8 > bool 1000000)\[1h:\]

This says to calculate the given expression as a range vector over the past hour, using the default (i.e. global) evaluation interval. From there it's a standard `avg_over_time`:

avg\_over\_time((rate(node\_network\_receive\_bytes\_total\[5m\]) \* 8 > bool 1000000)\[1h:\])

This is handy for ad-hoc usage, however if this is going to be used regularly by a rule it'd be best to use recording rules rather than subqueries. This is to avoid re-computing the entire hour worth of subquery output at every evaluation interval.

_Have PromQL questions? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
