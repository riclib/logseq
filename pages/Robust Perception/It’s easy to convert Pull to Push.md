---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/its-easy-to-convert-pull-to-push
author: [[Brian Brazil]] 
---
# It’s easy to convert Pull to Push

> If you have to choose one of push or pull in your core, which should it be?

---
If you have to choose one of push or pull in your core, which should it be?

I'm not going to get into the pros and cons of push and pull in this article, but instead look at what the best way is to go about supporting both. I'm going to presume that we're talking about transferring [metrics rather than events](https://www.robustperception.io/which-kind-of-push-events-or-metrics/). As an application author you'd often be asked to support a myriad of different monitoring systems, which will usually be a mix of both push and pull.

Let's say we choose push as our base. We'd have some way to register where to push the data to, and a regular interval push out the current state. At the code level an example would be [Telegraf's Output interface](https://github.com/influxdata/telegraf/blob/573bd4aa322ebcdbf6d3caf83fe4b4938e3eda27/output.go), and at the system level there are things like [Consul's telemetry support](https://www.consul.io/docs/agent/options.html#telemetry).

So that's push handled. How do we handle the pull? Well, pull wants the data to be collected when the pull happens. Which you can't do with a push system. So what you usually do is cache the last push, and return the stale data when a pull happens. This usually messes up the time stamps of the metrics, and we'd often need to also implement some form of GC to remove old data. This kinda works, but it's clunky and far from perfect.

So instead, lets try pull as our base. We'd have a function or endpoint which can return the current state. An example at the code level would be [Dropwizard's Reporters](http://metrics.dropwizard.io/3.2.3/manual/core.html#reporters), and on the system level anything exposing a Prometheus /metrics endpoint over HTTP.

How well does this handle push? Pretty easily. We'd setup a regular process to a pull the current state and then push it out. For example at the code level [Dropwizard](http://metrics.dropwizard.io/3.2.3/manual/graphite.html) and [Prometheus](https://github.com/prometheus/client_java#graphite) [clients](https://github.com/prometheus/client_python#graphite) support pushing to Graphite out of the box. A systems level example would be many Telegraf inputs, including the [Prometheus input plugin](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/prometheus).

Converting push to pull is tricky and imperfect, but converting pull to push is easy and has the right semantics. So if you have to choose, choose pull for your metrics API.
