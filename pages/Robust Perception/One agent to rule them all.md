---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/one-agent-to-rule-them-all
author: [[Brian Brazil]] 
---
# One agent to rule them all

> Another not uncommon question we get about Prometheus is as to why we don't have a single per-machine agent that handles all the collection, and instead have one exporter per application. Doesn't that make it harder to manage?

---
Another not uncommon question we get about [Prometheus](https://prometheus.io/) is as to why we don't have a single per-machine agent that handles all the collection, and instead have one exporter per application. Doesn't that make it harder to manage?

If you're starting from a low level of [infrastructure](http://www.robustperception.io/do-you-have-basic-infrastructure/) without a [machine/service database](http://www.robustperception.io/you-look-good-have-you-lost-machines/) having a single on-host daemon that gets information from all your applications may seem appealing. You'd have a file listing all the exporters/applications/subsystems and it'd go request them on every scrape. Then you just need Prometheus to come scrape it!

This presents a few problems that you'll hit as you scale.

The first is that this daemon would be a bottleneck operationally. Every time a new service is added to a machine along you'll need to alter central configuration management to so that the daemon knows about it. Similarly when services are removed. In the worst case you'd need to send a ticket to the central machine management team, blocking your launch until they got around to it.

The second problem is a lack of isolation between the various services. The default Prometheus scrape timeout is 10 seconds. What happens when one of the services is slower than that? Does the whole scrape fail? If not, how do you report this partial failure? What happens if one service starts spewing massive amounts of samples causing performance issues? These can be mitigated, but isolation is one of those things where it's troublesome to catch all the potential failure modes.

The third and final problem is that you won't be able to take advantage of one of the big wins of Prometheus, and that's the ability to think in terms of services instead of machines. If everything is coming from one per-machine agent then you're forced to think in terms of machines. You can't scrape just the MySQL, or just the Node Exporter - you have to take all the samples from everything at the same scrape interval. This would practically speaking constrain you to one centralised set of Prometheus servers controlled by one team, and preventing you from considering empowering each team to choose the monitoring that works best for them and running that monitoring in their own Prometheus server.

How could you do things to avoid this? Follow the path that Prometheus has laid down. Each target is its own entity which can be separately scraped, and all the targets can be found via service discovery. If you need an exporter, it's typically only a small bit more config to turn it up/down with the rest of the service. This avoids bottlenecks through decentralisation, provides a reasonable level of isolation and offers service-based monitoring that scales as you do!
