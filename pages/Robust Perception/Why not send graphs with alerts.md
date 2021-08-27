---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/why-not-send-graphs-with-alerts
author: [[Brian Brazil]] 
---
> You may have noticed that notifications from the Alertmanager are text. Wouldn't it be nice if Prometheus sent graphs along?

# Why not send graphs with alerts?


You may have noticed that notifications from the Alertmanager are text. Wouldn't it be nice if Prometheus sent graphs along?

One of the key features of Prometheus is labels. You can have a single alerting rule in one Prometheus that sends an alert for each of your thousands of machines distinguished by their labels. On top of that you could have a Prometheus in each of your datacenters sending in their alerts too. Alerts are sent on every evaluation for reliability. Let's say this amounts to a thousand alerts per second in aggregate. If you were to include a graph image of 100KB with each of those alerts that'd be 80Mbit/s of network traffic directed towards your Alertmanager.

That's just from a thousands alerts per second, which is not exactly a small number, but when all of your company is using the same Alertmanager to get the full benefits of grouping, throttling, and silencing it's not hard for things to add up to more bandwidth than is workable.

Aside from bandwidth, there is another issue. What graph would you send? Prometheus can't do much automatically from the alert expression as the power of PromQL means alerts aren't limited to just simple thresholds. Prometheus only knows about an alert when it starts firing. Usually you'd want a bit more history than that in your graph, so you'd have to define an extra expression of interest for the graph in each of your alerting rules.

You already have done all of this work though for your dashboards, so rather than duplicating it in your alerting rules you might include a link to your dashboard in your notifications or a link to an alert playbook that links to multiple relevant dashboards. This doesn't have bandwidth concerns, gives you up to date information, and includes other graphs that can be of use.

_Looking for advice on good alerting practices? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
