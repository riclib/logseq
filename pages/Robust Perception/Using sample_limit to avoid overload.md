---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/using-sample_limit-to-avoid-overload
author: [[Brian Brazil]] 
---
> Worried that your application metrics might suddenly explode in cardinality? sample_limit can save you.

# Using sample_limit to avoid overload


Worried that your application metrics might suddenly explode in cardinality? `sample_limit` can save you.

Like many things in life, labels are great in moderation. When a label with user's Id or email address is added to a metric though it is not likely to end well, as suddenly one of your targets could be pumping out hundreds of thousands of time series on every scrape causing performance issues with Prometheus.

You can try to figure this out after the fact using the approach in Which are my biggest metrics? to find the culprit, but it'd be nice to avoid the performance issues in the first place.

`sample_limit` is a scrape config field that will cause a scrape to fail if more than the given number of time series is returned. It is disabled by default. So for example:

scrape\_configs:
 - job\_name: 'my\_job'
   sample\_limit: 5000
   static\_configs:
     - targets:
       - my\_target:1234

Would cause the scrape of the target to fail if there were more than 5000 time series returned, with `up` being set to `0` as if the target was down. From there you can use [Dropping metrics at scrape time with Prometheus](https://www.robustperception.io/dropping-metrics-at-scrape-time-with-prometheus/) to remove the offending metric, while you're waiting for the application to be fixed. That is to say that `metric_relabel_configs` applied before `sample_limit`.

This feature is intended as an emergency release valve for a sudden increase in cardinality, if you were to try to implement a form of quota system using it you'd end up spending a lot of time micro-managing for little gain. Service discovery can return more targets too for example.

There are two metrics that will be of use if you are using this feature. The first is `scrape_samples_scraped`, which is the number of samples that was scraped. This will be set even if `sample_limit` kicks in and causes the scrape to fail. The second is `scrape_samples_post_metric_relabeling` which as the name indicates is how many samples are left after relabelling, and is the same value that `scrape_limit` uses. These can help you spot targets that might run into the limit before they actually hit it.

_Want help scaling Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
