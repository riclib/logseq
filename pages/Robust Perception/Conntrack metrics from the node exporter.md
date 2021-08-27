---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/conntrack-metrics-from-the-node-exporter
author: [[Brian Brazil]] 
---
> The node exporter includes metrics about the Linux connection tracking tables.

# Conntrack metrics from the node exporter


The node exporter includes metrics about the Linux connection tracking tables.

As metrics go, the conntrack ones don't seem very exciting. Many machines won't even have the `nf_conntrack` module loaded into the kernel. There's just two metrics:

\# HELP node\_nf\_conntrack\_entries Number of currently allocated flow entries for connection tracking.
# TYPE node\_nf\_conntrack\_entries gauge
node\_nf\_conntrack\_entries 205
# HELP node\_nf\_conntrack\_entries\_limit Maximum size of connection tracking table.
# TYPE node\_nf\_conntrack\_entries\_limit gauge
node\_nf\_conntrack\_entries\_limit 262144

One is how big the conntrack table can be, and the other is the number of current entries. These numbers are from my home router, `max_over_time(node_nf_conntrack_entries[365d])` is only coming to 2.5k so there's little to worry about.

So what is conntrack and why might it matter? If you're doing source-NAT or any form of firewalling that depends on thinking in terms of connections rather than merely packets then you need a way to link packets to connections - which is what the conntrack tables do. You can view the current table by running `conntrack -L`.

If you've more active connections to track than you have memory to track them, then that's bad. This sort of failure is what is often suspected when your home internet connection gets a bit dodgy after the router has been running for a few weeks/months. With these metrics though you can watch for this problem on your Linux routers.

_Have questions on network monitoring? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
