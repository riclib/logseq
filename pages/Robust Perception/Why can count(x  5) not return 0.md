---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/why-can-countx-5-not-return-0
author: [[Brian Brazil]] 
---
> When using the count aggregation operator you may have noticed that it sometimes returns nothing rather than 0. Why is this?

# Why can count(x > 5) not return 0?


When using the `count` aggregation operator you may have noticed that it sometimes returns nothing rather than 0. Why is this?

To explain, let's start with an example instant vector:

x{l="foo"} 2
x{l="bar"} 4

If you were to evaluate `count(x)` you would get `{}: 2`, which is to say a single sample with no labels and the value 2. This is what you'd expect.

If you were to evaluate `count by (l)(x)` you would get an instant vector with two elements, `{l="foo"}: 1` and `{l="bar"}: 1`. This all seems fine so far.

Now what if you do `count by (l)(x > 3)`? This makes the result a single sample of `{l="bar"}: 1`. This is as the sample with `l="foo"` would be filtered away, and the `count` aggregator would only be applied to the `l="bar"` sample. `count` can't invent a label out of nowhere after all.

The same applies to `count(x > 5)`. The instant vector returned by `x > 5` is empty so the result of the `count` is also going to be an empty instant vector.

The good news is that there is a way to get a 0 in this situation, by taking advantage of the `bool` modifier of comparison operators. Unlike normal comparison operators which filter if the comparison fails, the `bool` modifier will  return a 0 if the comparison fails and a 1 if it succeeds. Then you can add these up using `sum`.

So `sum(x > bool 5)` would return `{}: 0`. Similarly `sum by (l)(x > bool 3)` would return an instant vector with two elements, `{l="foo"}: 0` and `{l="bar"}: 1`.

_Want expert help with PromQL? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
