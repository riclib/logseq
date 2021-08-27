---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/alerting-on-approaching-open-file-limits
author: [[Brian Brazil]] 
---
> In a previous post we looked at dealing with reaching the open file limit. How about alerting before it happens?

# Alerting on approaching open file limits


In a previous post we looked at [dealing with](https://www.robustperception.io/dealing-with-too-many-open-files) reaching the open file limit. How about alerting before it happens?

`process_open_fds` and `process_max_fds`  are two metrics that many Prometheus client libraries produce out of the box, which are the current number of used file descriptors and the limit. With these it's easy to write an alert when a process hits 80% of the limit:

groups:
- name: example
  rules:
  - alert: ProcessNearFDLimits
    expr: process\_open\_fds / process\_max\_fds > 0.8
    for: 10m

File descriptors tend to be well below their limit, so by the time it is hitting 80% something is more than likely wrong.

This alert applies across all of your targets, as there are no limits on labels such as `job` or `instance`. This is a rare example of an alert that makes sense across all types of applications, as file limits are a pretty universal hard limitation - unlike say CPU which varies in importance.

This alert also shows a of the benefits of the Prometheus practice of using labels to distinguish time series from different types of applications, rather than prefixing the metrics with the name of the application. `process_open_fds` is `process_open_fds` no matter what application it is coming from, allowing for easy analysis across all applications that expose it.

_Want to improve application reliability? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
