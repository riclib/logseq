---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/push-needs-service-discovery
author: [[Brian Brazil]] 
---
# Push needs Service Discovery

> It's often claimed that an advantage of push-based monitoring systems is that, compared to pull-based systems like Prometheus, they don't need service discovery. This isn't true, and I'm going to explain why.

---
It's often claimed that an advantage of push-based monitoring systems is that, compared to pull-based systems like Prometheus, they don't need service discovery. This isn't true, and I'm going to explain why.

Firstly we need to clearly state which type of push and pull I'm talking about, as there's several variants. Here who makes the TCP connection isn't really relevant, nor is it relevant if it's events or metrics that are being transferred.

The question is whether the target being monitored decides to registers with the monitoring system, or whether the monitoring system already knows what targets should be monitored. This can be thought of as as bottom-up versus top-down.

The problem is thus: if you have bottom-up service discovery and a target never registers, how would you know?

You wouldn't.

You could have [lost servers](https://www.robustperception.io/you-look-good-have-you-lost-machines/) which are laying about [wasting money](http://blog.runnable.com/post/153498635761/how-we-saved-98-on-infrastructure-monitoring). What's worse, they could cause an outage if they unexpectedly reawaken and are still running old code.

To handle this in a push-based system like Graphite you need a separate process to reconcile who is pushing against who is meant to be pushing from a top-down service discovery system. This is often going to be a machine database, service database, or a scheduler like Kubernetes/Marathon. There is flexibility in how often you need to do this. Depending on how dynamic and automated your infrastructure is, anywhere from once a minute to once a quarter may make sense.

With a pull-based system like Prometheus this is something you may already have. Prometheus requires some form of service discovery to work in the first place, which is often a top-down source such as Kubernetes. From there it's a matter of using `[up](https://www.robustperception.io/whats-up-doc/)` in an alert. However not all service discovery mechanisms supported by Prometheus are top-down. Some like Consul are bottom-up though, so you'd still need that reconciliation process happening in the background.

So it's not that push-based monitoring systems don't need service discovery, it's that push-based monitoring systems don't need realtime top-down service discovery to find their targets. Push-based monitoring systems still need top-down service discovery.

Why?

Because all monitoring systems need top-down service discovery.

(I should also mention Paul Dix's post on [InfluxDB adding pull support to Kapacitor](https://www.influxdata.com/monitoring-with-push-vs-pull-influxdb-adds-pull-support-with-kapacitor/), where he points out that in push-based monitoring the targets also need service discovery in order to figure out where to push to.)
