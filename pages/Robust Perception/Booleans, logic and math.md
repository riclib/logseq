---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/booleans-logic-and-math
author: [[Brian Brazil]] 
---
# Booleans, logic and math

> Prometheus doesn't have an explicit boolean type or functionality. However there is a convention and enough power in PromQL to work with booleans.

---
[Prometheus](https://prometheus.io/) doesn't have an explicit boolean type or functionality. However there is a convention and enough power in PromQL to work with booleans.

By convention in Prometheus, boolean metrics have either the value 0 for false or 1 for true. The most well known example is [`up`](https://www.robustperception.io/whats-up-doc/), but there are others like the blackbox exporter's `probe_success`.

Alerting on these is quite common and easy, for example:
```yaml
groups:
- name: test.rules
  rules:
  - alert: InstanceDown
    expr: up{job="node"} == 0
    for: 10m
```
This will alert for each node which is down, and thus has an `up` of 0.

Sometimes you want to do something more sophisticated, and the standard [boolean algebra](https://en.wikipedia.org/wiki/Boolean_algebra) operators are needed.

How about checking if at least one node is up? `max by (job)(up)` will return 1 if at least one value is 1, and 0 otherwise. `max` behaves as an OR operator.

Similarly for checking if all nodes are up,  `min by (job)(up)` will return 0 if at least one value is 0, and 1 otherwise. `min` behaves as an AND operator.

We have AND and OR, what about NOT? `1 - up` will return the negation, as 1 - 0 = 1 and 1 - 1 = 0.

With AND, OR, and NOT we can build NAND and NOR, from upon all other boolean operators can be built.

We can do a little better for XOR, though there's two interpretations of how to deal with more than two inputs. If you mean the multi-input XOR where exactly one input must be true, then `sum by (job)(up) == bool 1` will provide that. For the multi-input XOR where an odd number of inputs must be true, then `sum by (job)(up) % 2 == bool 1`.

These take advantage of the `bool` modifier on comparisons. Usually a comparison in PromQL will filter results, returning only the ones that match. With `bool` instead you get a 0 or 1 result for every set of operands, indicating if they matched.

This can be used in a variety of ways. NOT could also be implemented as `up == bool 0`. If you wanted a boolean indicating if at least 3 nodes were up you could use  `sum by (job)(up) >= bool 3`.

These examples are all within one metric. What if we want to do boolean algebra between two metrics `a` and `b`? AND is `a * b`, OR is `a + b > bool 0`, and XOR is `a + b == bool 1`.

As PromQL supports floating point numbers, we can also do math that would be tricky with booleans alone. For example we can use `avg` to calculate the ratio of up nodes, and alert if too high a proportion are down:
```yaml

groups:
- name: test.rules
  rules:
  - alert: InstancesDown
    expr: avg(up{job="node"}) BY (job) < 0.75
    for: 10m
```
None of the behaviour above was added to PromQL with booleans in mind, rather it is taking advantage of the richness of PromQL combined with a little bit of computer science knowledge.

It's not often that you'll need full boolean algebra, but you can be confident that it'll be possible when it does come up.
