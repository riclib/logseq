---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/why-info-style-metrics-have-a-value-of-1
author: [[Brian Brazil]] 
---
> You've seen metrics like prometheus_build_info, but why do they have a value of 1?

# Why info-style metrics have a value of 1


You've seen metrics like `prometheus_build_info`, but why do they have a value of 1?

Since the machine roles post came out [all the way back in 2015](https://www.robustperception.io/how-to-have-labels-for-machine-roles), this has become a standard pattern within the Prometheus ecosystem. Such metrics these days are known as info metrics and usually have an `_info` suffix.

Something that has never really been discussed is why the value is always 1. Why not some other value?

The answer is that 1 is the [identity element](https://en.wikipedia.org/wiki/Identity_element) for multiplication, which is to say that if you multiply a number by 1 you get the same number back. As `group_left` needs to be associated with a binary operator we need an operator and operand that will let us do what we need to with labels, without affecting the value of the main expression.

There are other operators each with their own identity elements, for example addition and 0. Why not use those? While that would work fine in terms of labels, it's a little less useful when it comes to working with the metric on its own. For example if you want to count how many of each version of Prometheus is running you can do `count by (version)(prometheus_build_info)` no matter what the value of the info metric is and subsequent aggregations can use `sum` on top of that. If however the value is 1 you can use `sum` for both the initial count and all subsequent aggregations, which is simpler and thus you're less likely to make a mistake when constructing PromQL expressions using it.

So why is the value 1? Math and simplicity.

_Have PromQL questions? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
