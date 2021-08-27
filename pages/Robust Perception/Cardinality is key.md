---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/cardinality-is-key
author: [[Brian Brazil]] 
---
> Prometheus performance almost always comes down to one thing: label cardinality.

# Cardinality is key


Prometheus performance almost always comes down to one thing: label cardinality.

Cardinality is how many unique values of something there are. So for example a label containing HTTP methods would have a cardinality of 2 if you had only GET and POST in your application.

So if your application started supporting PUT, then the cardinality would be 3. Nice small numbers, so where's the problem? HTTP method is likely only one dimension of interest, you likely also have multiple HTTP paths, different machines running this code, a development environment, and more than one datacenter. On top of that if using a histogram then that has a cardinality of 12 out of the box.

All of these cardinalities end up being multiplied together, and each time series in the resultant product is one more that needs to be ingested on every scrape, stored, indexed, and processed for queries. On the plus side the product is likely sparse, as every possible combination is unlikely to happen, however that's little comfort in practice as it only delays the inevitable combinatorial explosion.

It's fairly common that things start out reasonable. You might have a histogram covering 2 HTTP methods, 7 HTTP paths, 5 machines, and a Prometheus typically only monitors one environment and datacenter. So that's 2x7x5x12 = 840. Well within the capabilities of a single Prometheus.

What tends to catch you out is that things usually don't grow in only one dimension. Increased traffic means more machines, and more users usually means more features so new endpoints. So you might now have say 3x8x6x12, which is an increase of just 1 for each of the first three factors, resulting in 1728 or more than double the original!

It's still small overall, but this is just one metric, from one subsystem, and this is only one minor growth spurt. Over time growth accumulates and compounds, and can bring you to a point where gradually your Prometheus starts to creak. No one change caused it, but it still needs to be dealt with before your monitoring falls over. A Prometheus 2.x can handle somewhere north of ten millions series over a time window, which is rather generous, but unwise label choices can eat that surprisingly quickly.

Some particular things to watch out for are breaking out metrics with labels per customer. This usually works okay when you've tens of customers, but when you get into the hundreds and later thousands this tends not to end well. Increasing the number of buckets in your histograms also tends to go sour, as histograms are often also broken down by other labels so the growth of both combines. It's not at all unusual that over half the resource usage of a Prometheus is due to less than ten metrics, and moving the label values into the metric name doesn't make a difference - it just makes your life harder!

As a general rule of thumb I'd avoid having any metric whose cardinality on a /metrics could potentially go over 10 due to growth in label values. The way I think of it is that if it has already grown to be 10 today, it might be 15 in a year's time. It can be additionally okay to have no more than a literal handful within a given Prometheus that go to 100. These are only rules of thumb, if you know for example that the number of replicas of your service will always be tiny then you can exceed these a bit, conversely if you're expecting thousands of replicas I'd carefully consider every time series on the /metrics.

If you can't get such granular data in a metrics based monitoring system, doesn't that make it kinda useless? Not really, metrics based monitoring is one set of tradeoffs. It gives you real time information (and real time is expensive as a general rule) about your system in broad strokes, but has cardinality limitations. Complementing metrics are event logs, which tend to be more delayed and have fewer pieces of information, but no cardinality limitations.

So taking the above examples you may have a latency histogram broken out only by HTTP endpoint, from which you can calculate a rough percentile good enough to alert on from your metrics system. When a human gets to look at it (it takes a few minutes to fully wake up after all) your logs pipeline will have by then have processed the logs for the relevant time period, and combining logs and metrics you can narrow down what the issue is. Metrics giving you a high level view of your subsystems, logs the blow-by-blow of individual requests.

_Wondering how to scale your monitoring? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
