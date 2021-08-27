---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/target-labels-not-metric-name-prefixes
author: [[Brian Brazil]] 
---
> Services are not distinguished by their metric names in Prometheus.

# Target labels, not metric name prefixes


Services are not distinguished by their metric names in Prometheus.

With the dotted.string.style of metrics it's usual to prefix metrics with the host they're on, environment, datacenter, application, and so on. This is however not the way to handle things in Prometheus whose labels provide a more powerful data model. In addition Prometheus has a top-down way of handling service discovery.

With Prometheus instrumentation it'd be very unusual for all the metrics on a /metrics to share a prefix. There's typically some `process_` metrics, runtime-specific metrics like `go_` or `jvm_`, various libraries, and then metrics for the application itself. Fundamentally a metric name is not about applications or deployments, but a pointer to a specific line of code in a library of some form. In the case of a `main()` function or equivalent there's no difference between the application and the "library", but that's not the case for other metrics.

What this enables is that you can aggregate library metrics across different applications. For example the maintainer of the RPC library may wish to know if it makes sense to adjust compression defaults, or if the average request size is sufficient to justify a certain optimisation. This general idea is known as horizontal monitoring. The developer/deployer of an application is not the only person who may wish to monitor it.

How do you distinguish the `process_cpu_seconds_total` from the applications you care about from the ones you don't? The answer in Prometheus is target labels. At a minimum you'll have a [`job` label](https://www.robustperception.io/what-is-a-job-label-for) to indicate the type of thing you're monitoring, and an [`instance` label](https://www.robustperception.io/controlling-the-instance-label) to identify the specific process. You might also have others for the [environment, datacenter, team](https://www.robustperception.io/target-labels-are-for-life-not-just-for-christmas) etc. When using PromQL you can (and should) restrict down which time series you're interested in using selectors such as `my_metric{job="myjob"}`.

An additional benefit of this design is that you don't need to change the metric names in the code when the deployment strategy changes. Such as if you bring up a 2nd set of caches with a completely different purpose and performance profile. You can likely reuse recording rules/alerts with some tweaking, plus reuse dashboards by taking advantage of Grafana's variables to allow the viewer to choose appropriate target labels. In this way a common library/binary could provide standard rules and dashboards for all of its users.

So if you're trying to prefix all the names on a /metrics, look to target labels instead.

_Unsure about Prometheus instrumentation? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
