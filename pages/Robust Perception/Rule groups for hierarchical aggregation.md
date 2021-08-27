---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/rule-groups-for-hierarchical-aggregation
author: [[Brian Brazil]] 
---
> Prometheus 2.0 brought with it rule groups, making hierarchical aggregation easier than ever.

# Rule groups for hierarchical aggregation


Prometheus 2.0 brought with it rule groups, making hierarchical aggregation easier than ever.

In Prometheus 1.x recording rules were all run concurrently. This meant that it wasn't safe for one rule to depend on another, as you could see either the result of the previous evaluation or of the current one. These race conditions meant that having dependencies between rules was risky, and best avoided.

With Prometheus 2.0 this is no longer the case, as rules within a rule group are executed sequentially. So you can now safely have rules depending on each other, within a rule group:

groups:
- name: node\_rules
  rules:
  - record: instance\_mode:node\_cpu:rate5m
    expr: sum without(cpu)(rate(node\_cpu{job="node"}\[5m\]))
  - record: mode:node\_cpu:rate5m
    expr: sum without(instance)(instance\_mode:node\_cpu:rate5m{job="node"})

Here we first aggregate up a per-instance per-mode CPU usage for a machine, and then get per-mode usage across all node exporters in the Prometheus server. This is more efficient as it saves the `rate()` having to be taken multiple times.

Rule groups are run concurrently, so it is not generally safe to have a rule in one rule group depend on another rule group.

_Need help with PromQL? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
