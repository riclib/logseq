---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/whats-up-doc
author: [[Brian Brazil]] 
---
# What’s “up” Doc?

> One of the advantages of pull-based monitoring such as Prometheus is that you can tell if the target is healthy as part of the scrape. How do you do that though?

---
One of the advantages of pull-based monitoring such as [Prometheus](https://prometheus.io/) is that you can tell if the target is healthy as part of the scrape. How do you do that though?

A not uncommon question that's asked about Prometheus is how to tell if a server is not responding or detect if samples are no longer being ingested from a target. The first tool of choice for simple blackbox monitoring is the [blackbox exporter](https://github.com/prometheus/blackbox_exporter), and you could use `unless` and `offset` to do edge detection to detect that a time series is not being updated. There is however a simpler way that covers most use cases.

When Prometheus scrapes a target, it doesn't just ingest the returned samples. It also adds in some additional samples about the scrape itself. The main one of interest is `up`, which will be 0 if the scrape failed or 1 if the scrape succeeded. I've previously looked at how to [alert](https://www.robustperception.io/alerting-on-down-instances/) on this.

There are a few things you should know about `up`. The first is that because it doesn't come from the scrape itself, `metric_relabel_configs` doesn't apply. That is to say that `up` always has the target's label, per service discovery and `relabel_configs`.

The second is that `up` will get new values for as long as the target is returned from service discovery. This means that if auto scaling removes an instance you won't get an alert that it's down, which is what you want. It also means however that if auto scaling removes all your instances, you won't get an alert either. To safeguard against this, it wise to create an alert with the expression `absent(up{job="myjob"})` for the case of all of the job disappearing.

The third is that there are [additional time series](https://prometheus.io/docs/concepts/jobs_instances/#automatically-generated-labels-and-time-series) that behave like `up`, which are all prefixed by `scrape_`. These can be useful for debugging. Why is it `up` rather than `scrape_up` you might ask? Well, `up` is so commonly used that it's worth a little inconsistency in the naming scheme.

The forth and final thing is that some exporters will succeed even if the instance they're hitting is down, usually because they can still provide some useful information in that case. Thus an alert on `up` being 0 is necessary but not sufficient. In this case you may also want an alert on `haproxy_up`, `mysqld_up`, `consul_up`, `probe_success` (used by the blackbox exporter) and so on.
