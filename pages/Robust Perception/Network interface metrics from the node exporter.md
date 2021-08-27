---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/network-interface-metrics-from-the-node-exporter
author: [[Brian Brazil]] 
---
> Along with many others, the node exporter exposes network interface metrics.

# Network interface metrics from the node exporter


Along with many others, the node exporter exposes network interface metrics.

Network interface metrics have the prefix `node_network_` on the node exporter's /metrics, and a `device` label. These are distinct from the `node_netstat_` metrics which are about the kernel's network subsystem in general.

The main metrics you're probably looking for are bandwidth usage which comes from the `netdev` module, and is covered by `node_network_receive_bytes_total` and `node_network_transmit_bytes_total`. As bytes are the base unit, if you want bits per second you can multiply by 8 in your graphs such as `rate(node_network_transmit_bytes_total[5m]) * 8`.

There's also other counter metrics from `/proc/net/dev` including `packets`, `errs`, `drop`, `fifo`, `compressed`, and `multicast` for both `receive` and `transmit`. `colls` (collisions) and `carrier` are in addition present only for transmit. To calculate the transmission error ratio you could use

  rate(node\_network\_transmit\_errs\_total\[5m\]) 
/ 
  rate(node\_network\_transmit\_packets\_total\[5m\])

however this may or may not include unsuccessful sends in the denominator. As with many things in the kernel the exact meaning of these counters will vary, and what [documentation](https://www.kernel.org/doc/Documentation/ABI/testing/sysfs-class-net-statistics) there is isn't particularly illuminating so you'd need to dig through the source for the relevant driver to be sure. And even then you have to hope that this is not one of those things that also vary by hardware model/firmware version, nor that varies by kernel version. (This is incidentally a prime example of why a given metric should live entirely inside a single class/file. To share between different implementations you need a good specification that everyone follows so the metric always has the same meaning, otherwise it's not the same metric.)

In practice you can usually make your monitoring good enough without worrying about these details where metrics are unavoidably underspecified. For example if your level of errors is so high that including/excluding of errors in the denominator is significant then your graphs would have indicated that there's something wrong long before you reached that point. More than 100% errors is obviously a problem after all, and if you over-corrected in the other direction then at worst you'd be halving the true error ratio and 5% errors is a big a problem as 10%.

There's also the `netclass` module, which reads from `/sys/class/net/`. The documentation is on [kernel.org](https://www.kernel.org/doc/Documentation/ABI/testing/sysfs-class-net). `node_network_mtu_bytes` and `node_network_speed_bytes` are self explanatory. `node_network_info` has various string information about the interface and its state:

node\_network\_info{address="00:00:00:00:00:00",
    broadcast="00:00:00:00:00:00",device="lo",duplex="",
    ifalias="",operstate="unknown"} 1
node\_network\_info{address="02:52:48:00:7b:aa",
    broadcast="ff:ff:ff:ff:ff:ff",device="docker0",duplex="",
    ifalias="",operstate="down"} 1
node\_network\_info{address="3c:31:f7:91:f2:93",
    broadcast="ff:ff:ff:ff:ff:ff",device="eth0",duplex="full",
    ifalias="",operstate="up"} 1

`node_network_up` is 1 if `operstate` is `up`. `node_network_dormant` and `node_network_link_mode` can be used to tell if something like [802.1X](https://en.wikipedia.org/wiki/IEEE_802.1X) authentication is needed for the interface to be usable.

If you want to check if your link has been flapping there's `node_network_carrier_changes_total`, `node_network_carrier_down_changes_total` and `node_network_carrier_up_changes_total`. As these are counters you don't have to worry about missing a flap in `node_network_carrier` that happens between scrapes.

`node_network_iface_id`is the same as SNMP's `ifIndex`, though if you're using the node exporter you should have no need for SNMP.  `node_network_address_assign_type` indicates where the MAC address came from e.g. hardware, random, or set. `node_network_name_assign_type` is where the name of the interface came from - did you know that [eth0 is no longer a thing by default](https://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/)?

`node_network_protocol_type` is the type of interface, e.g. [ethernet, loopback, PPP](https://github.com/torvalds/linux/blob/ae4b064e2a616b545acf02b8f50cc513b32c7522/include/uapi/linux/if_arp.h). `node_network_flags` is a bitmask for flags like [broadcast, multicast, promiscuous](https://github.com/torvalds/linux/blob/ae4b064e2a616b545acf02b8f50cc513b32c7522/include/uapi/linux/if.h#L82-L108) etc.

As with most metrics, many of these are only of interest for after-the-fact debugging and are not something you'd put on a dashboard or wake someone up based on.

_Have questions about network monitoring? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
