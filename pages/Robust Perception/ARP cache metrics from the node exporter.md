---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus
source: https://www.robustperception.io/arp-cache-metrics-from-the-node-exporter
author: [[Brian Brazil]]
---
> The node exporter has metrics about the ARP table.

# ARP cache metrics from the node exporter


The node exporter has metrics about the ARP table.

The ARP cache is part of how computers figure out which IPv4 addresses match up with which MAC or hardware addresses, so that packets can be efficiently switched on local network segments. The contents of this cache can be seen in `/proc/net/arp`, or in a more human readable form by running `ip -4 neigh` or `arp -n`.

The node exporter exposes the number of entries in the ARP cache, split out by device:

\# HELP node\_arp\_entries ARP entries by device
# TYPE node\_arp\_entries gauge
node\_arp\_entries{device="br0"} 19

For the vast majority of use cases the size of this cache is uninteresting, as when there are problems with ARP it's usually the fact that bad state has gotten into the ARP cache that is relevant than the number of entries in the cache. If however you had a network segment with a lot of devices, you could use this to ensure that your cache wasn't too small and getting too near the limit in `/proc/sys/net/ipv4/neigh/default/gc_thresh3` which defaults to 1024.

_Have questions on network monitoring? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
