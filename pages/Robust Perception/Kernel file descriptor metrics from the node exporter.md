---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/kernel-file-descriptor-metrics-from-the-node-exporter
author: [[Brian Brazil]] 
---
> The node exporter provides kernel file descriptor metrics.

# Kernel file descriptor metrics from the node exporter


The node exporter provides kernel file descriptor metrics.

The kernel file descriptor metrics are pretty simple, there's only two of them which come from [file-nr and file-max](https://www.kernel.org/doc/Documentation/sysctl/fs.txt):

\# HELP node\_filefd\_allocated File descriptor statistics: allocated.
# TYPE node\_filefd\_allocated gauge
node\_filefd\_allocated 17843
# HELP node\_filefd\_maximum File descriptor statistics: maximum.
# TYPE node\_filefd\_maximum gauge
node\_filefd\_maximum 3.273286e+06

They mean what you'd think they mean, the kernel has a limit for how many file descriptors everything in the system can simultaneously have allocated. This is based on how much memory your machine has by default, and running out would be bad. It is however extremely unlikely that you'd ever run in to this limit.

Where there is confusion is that the `node_filefd_*` metrics are machine-level kernel metrics - as you'd expect from the node exporter. If you're worried about running out of file descriptors, what you almost certainly want are the [per-process file descriptor metrics](https://www.robustperception.io/dealing-with-too-many-open-files) `process_open_fds` and `process_max_fds` instead.

I've seen this cause confusion for users, and not just of Prometheus. Unless you're running a system that's expected to e.g. be handling millions of concurrent network connections such that this limit is actually a potential concern, there's no need to have `node_filefd_allocated` (or `file-nr` from sar etc.) on a dashboard.

_Wondering what you should be alerting on? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
