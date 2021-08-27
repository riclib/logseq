---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/dont-federate-instance-labels
author: [[Brian Brazil]] 
---
> Federation can be quite useful, but it's not replication.

# Don’t federate instance labels


Federation can be quite useful, but it's not replication.

We've [previously looked](https://www.robustperception.io/federation-what-is-it-good-for) at how federation can be usefully employed, and the downsides of using it otherwise. In this article I'll provide you with a rule of thumb for spotting where the use of federation is most likely inappropriate.

Very simply, when federating if the time series produced include `instance` labels then the usage of federation is probably not appropriate.

To quickly recap federation works well when it's used to pull in a limited set of aggregated time series. If the time series you are pulling in via federation contain `instance` labels that indicates that firstly they aren't aggregated as the usual first step in aggregation of aggregating the `instance` label away hasn't happened. For example when pulling series from a datacenter Prometheus to a global Prometheus, the global Prometheus should care about overall performance at the datacenter level rather than what each individual target is up to.

Secondly it's likely not a limited set of time series as time series with `instance` labels include the raw series that come directly from targets, so it's probable that you're pulling in entire targets at once rather than even a subset of those targets' series.

I'm aware of only one general use case where federating series with an `instance` label makes sense, and that is in a [horizontal sharding](https://www.robustperception.io/scaling-and-federating-prometheus) setup. In such a setup targets are spread arbitrarily across a set of scraping Prometheus servers, and the top level Prometheus pulls in aggregated series from those via federation. In such a setup you often wish to jump to the scraping Prometheus that handles a particular target, and for that federating one single series (such as `up`) to the top level Prometheus makes it possible to know which scraper has each target without having to check all of them. As it is only a single series per target, you only care about the labels, and the value of the series isn't used, the performance and semantic issues that'd usually apply to federating instance-level series are unlikely to come up.

Using federation inappropriately is one of the biggest reasons that users run into performance problems with their Prometheus architecture, so try to avoid getting into that situation in the first place.

_Have questions about architecting Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
