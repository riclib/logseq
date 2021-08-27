---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/dont-cross-the-screams-monitoring-across-failure-domains
author: [[Brian Brazil]] 
---
> Scraping targets across datacenters will make things better, right?

# Don’t cross the screams: Monitoring across failure domains


Scraping targets across datacenters will make things better, right?

If a datacenter goes down, it'd seem useful to have monitoring of its targets from outside it. That way even though the Prometheus within the malfunctioning datacenter may be broken, you'd still get metrics.

This is enticing, but generally not useful in practice. External monitoring of a datacenter involves additional network hops, so will be less reliable - in addition to slower and likely more expensive. You'd then have a view with the datacenter that works generally, and from outside that's about the same but with more failed scrapes. As in all cases when there's more than one Prometheus, there is the challenge of how to synthesise that into a single view.

There's a more basic issue. Such a setup pre-supposes that it'll help in cases where the Prometheus inside the datacenter can't scrape a target, but a Prometheus outside the datacenter can. Network failures like that would be quite odd, and I'd expect far exceeded by cases where either noone can scrape the targets (e.g. complete power failure) or the outside Prometheus cannot (e.g. network connectivity failure, outside datacenter power failure).

In terms of overall robustness, having a 2nd Prometheus inside the datacenter on a different power bus and network switch is going to give you better value if this concerns you.

This is not the say that a Prometheus should never scrape across datacenters. It makes sense to scrape a Prometheus inside the datacenter to know if the datacenter/Prometheus is working, and similarly for related network connectivity checks. You would also scrape across datacenters when [federating for a global Prometheus](https://www.robustperception.io/federation-what-is-it-good-for).

In general though, a Prometheus should scrape things that are local to it that share failure domains. This usually means on the same network segment/broadcast domain, in one datacenter, using the same power source, without any firewalls/NAT or whatnot in the way.

_Need advice on Prometheus architecture? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
