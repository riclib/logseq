---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/new-features-in-node-exporter-0-16-0
author: [[Brian Brazil]] 
---
> Node exporter 0.16.0 is out, and with some big changes. Let's take a look.

# New Features in Node Exporter 0.16.0


Node exporter 0.16.0 is out, and with some big changes. Let's take a look.

The Node exporter is one of the oldest exporters, and thus it doesn't always follow best practices that have emerged in the past few years. This is something that we've wanted to fix for quite some time, and it is finally the case with with 0.16.0. The bad news is that bringing the Node exporter in line means changing the names of many of the most important metrics. With these changes it is likely that many of your existing dashboards and alerts relating to the Node exporter will need updating, and we've also provided an updated version of our own [Host Stats](https://grafana.com/dashboards/6014) dashboard.

Some examples of the metrics changing include:

-   `node_cpu` ->  `node_cpu_seconds_total`
-   `node_memory_MemTotal` -> `node_memory_MemTotal_bytes`
-   `node_memory_MemFree` -> `node_memory_MemFree_bytes`
-   `node_filesystem_avail` -> `node_filesystem_avail_bytes`
-   `node_filesystem_size` -> `node_filesystem_size_bytes`
-   `node_disk_io_time_ms` -> `node_disk_io_time_seconds_total`
-   `node_disk_reads_completed` -> `node_disk_reads_completed_total`
-   `node_disk_sectors_written` -> `node_disk_written_bytes_total`
-   `node_time` -> `node_time_seconds`
-   `node_boot_time` -> `node_boot_time_seconds`
-   `node_intr` -> `node_intr_total`

A few of these such as data read/written to disk and how long those reads/writes took have changed units to base units such as bytes and seconds. With these changes the Node exporter's metrics are easier to understand, however as with most exporters they are still not 100% perfect due to the limitations of the input data and documentation we have to work with.

In addition to these changes, the output of some of collectors of the Node exporter have been trimmed. The vmstat and netstat collectors between them produce several hundred metrics about detailed Linux kernel internals. While usually the approach in the Prometheus ecosystem is to expose everything, several hundred additional metrics of highly detailed debug information from every individual machine in your fleet is pushing it a bit. If there are some of these metrics you want back these changes are only a default, so exposing them is only a command line flag away.

_Want to get the most out of machine monitoring? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
