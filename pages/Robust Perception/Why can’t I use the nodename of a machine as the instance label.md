---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/why-cant-i-use-the-nodename-of-a-machine-as-the-instance-label
author: [[Brian Brazil]] 
---
> The machine knows its own name, couldn't Prometheus use it?

# Why can’t I use the nodename of a machine as the instance label?


The machine knows its own name, couldn't Prometheus use it?

This is a not uncommon question about Prometheus and service discovery. If you run `uname -n` you'll see a machine's nodename, and this is something that'd be useful to have as part of your `instance` label. The node exporter even exposes it as the `nodename` label on the `node_uname_info` metric!

This is not something that Prometheus can do. The reason is that the target labels are determined by service discovery and relabelling, before Prometheus ever attempts to talk to a scrape target. You could in principle have the scrape target attach it as a label on every sample, but that goes against the top-down strategy that Prometheus uses for configuration management. Put another way, a target doesn't know how a scraper views it. For example the database team, application team, and infrastructure teams may all have different views on where a target belongs in their label taxonomy. One person's primary database is another's 4th machine in the 7th rack in the Irish datacenter.

Another potential way to do it would be to have targets pass back this information as some form of metadata. Ignoring all the complication that'd involve for every exporter and application out there, it doesn't work for a simpler reason: if Prometheus couldn't talk to the target it couldn't determine the target labels and thus we couldn't know what labels `up` should have.

This may seem like a bit of a downer, but while Prometheus can't do this itself that doesn't mean that the end goal of having the `instance` label match the nodename impossible. In any vaguely developed infrastructure there should be a working DNS system, with DNS records that match up with the nodenames for each machine. You'd be in for quite a lot of head scratching if this wasn't in place! So instead of providing an IP address, provide a DNS name to Prometheus and the nodename is [right there for you to access in relabelling](https://www.robustperception.io/controlling-the-instance-label).

By coming at the problem from the other direction, it's not so hard to deal with.

_Wondering how to organise your target labels? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
