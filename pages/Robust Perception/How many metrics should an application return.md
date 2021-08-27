---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/how-many-metrics-should-an-application-return
author: [[Brian Brazil]] 
---
> While each application is different, a rough idea of how many metric there should be would be useful.

# How many metrics should an application return?


While each application is different, a rough idea of how many metric there should be would be useful.

When starting out with instrumentation there's often uncertainty as to how many metrics and time series is too few or too much. I'd like to give some rules of thumb to help judge if the metric data volume is about right for a given type of application.

For very simple applications with little logic that only do one thing, I'd expect on the order of 100 time series. Caches are an example of systems that usually fall into this category, and in the Prometheus world the Pushgateway is another. You would typically have only a handful of metrics you have added, on top of the various metrics that the client library and any other dependencies you use provide out of the box. The Pushgateway has around 120 time series for example.

For complex applications with lots of moving parts, I'd expect on the order of 1000 time series. As an example the Prometheus server itself currently exposes somewhere around 700 time series, depending on which version and features you're using. This is as there's out of the box time series, plus all the metrics that have been added for the various subsystems.

When an application exposes more than that, getting up towards 10,000 time series that's an indication that you may have a cardinality issue and might want to cut back on labels a bit. This is however unavoidable sometimes for cases such as reverse proxies where there's many many backend services, or databases where there's many many tables and you need information for each. Be wary of going too much above 10,000, Prometheus is designed for many small scrapes, not a handful of massive ones.

Having said all that, this the above doesn't mean you should add new metrics just to reach the above number. This are just my personal experience of what various types of well instrumented services tend to expose, so having more or less than that doesn't automatically indicate a problem. It's a reference, not a target.

_Wondering how to manage cardinality? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
