---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/analyse-a-metric-by-kernel-version
author: [[Brian Brazil]] 
---
> Using PromQL you can combine metrics for analysis.

# Analyse a metric by kernel version


Using PromQL you can combine metrics for analysis.

It's not unknown that a new kernel version can introduce subtle problems, which can be difficult to spot on a machine by machine basis. Let's say that you suspected that something was up in relation to TCP sockets in the TIME\_WAIT state. The number of such sockets is covered by the `node_sockstat_TCP_tw` metric.

So you can do:

avg without (instance)(
    node\_sockstat\_TCP\_tw 
  \* on(instance) group\_left(release)
    node\_uname\_info
)

What this does is add the `release` label (the kernel version) to the `node_sockstat_TCP_tw` metric based on the instance label for `node_uname_info` being the same. Finally we average the values, ignoring the `instance` label - which should produce a per kernel version result.

This is useful only as long as all the kernel versions are seeing about the same load. Let's say we wanted to normalise by the number of in-use TCP sockets, we can do:

avg without (instance)(
    node\_sockstat\_TCP\_tw 
  / 
    node\_sockstat\_TCP\_inuse
  \* on(instance) group\_left(release)
    node\_uname\_info
)

While taking an average of a pile of ratios is dubious statistically, this could help spot at a very high level if there's a potential issue.

_Unsure how to do something in PromQL? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
