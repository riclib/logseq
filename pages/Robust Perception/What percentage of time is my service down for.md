---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/what-percentage-of-time-is-my-service-down-for
author: [[Brian Brazil]] 
---
> Have you ever wondered what percentage of time a given service or application spends up or down?

# What percentage of time is my service down for?


Have you ever wondered what percentage of time a given service or application spends up or down?

In this blogpost we'll demonstrate how to use the [Blackbox exporter](https://github.com/prometheus/blackbox_exporter) with [Prometheus](https://github.com/prometheus/prometheus) in order to achieve this.

Setting up a simple contrived example, we'll run both the Blackbox and Node exporter, and configure Prometheus to tell the Blackbox exporter to issue a simple HTTP probe to the [node exporter](https://github.com/prometheus/node_exporte) and scrape the result.

global:
  scrape\_interval:    5s
  evaluation\_interval: 5s

scrape\_configs:
  - job\_name: 'node'
    metrics\_path: /probe
    params:
      module: \[http\_2xx\]  # Look for a HTTP 200 response.
    static\_configs:
      - targets:
        - http://localhost:9100
    relabel\_configs:
      - source\_labels: \[\_\_address\_\_\]
        target\_label: \_\_param\_target
      - source\_labels: \[\_\_param\_target\]
        target\_label: instance
      - target\_label: \_\_address\_\_
        replacement: 127.0.0.1:9115  # The blackbox exporter's real hostname:port.

Using the query function [avg\_over\_time()](https://prometheus.io/docs/prometheus/latest/querying/functions/#aggregation-_over_time) we can get the average value of the blackbox exporter's probe\_success metric over a given time period which simply reports 1 or 0 depending on whether the target probed responds with a HTTP 200 response for our given probe.

The examples below show the result of this query function when looking at probe\_success over a period of 15 minutes. We multiply by 100 to get a percentage.

In order to get a percentage of 80%, I killed the Node exporter for a few minutes.

(The full query used is `avg_over_time(probe_success{job="node"}[15m]) * 100`

[![](https://www.robustperception.io/wp-content/uploads/2018/01/Screen-Shot-2018-01-31-at-15.37.38.png)](https://www.robustperception.io/wp-content/uploads/2018/01/Screen-Shot-2018-01-31-at-15.37.38.png)

[![](https://www.robustperception.io/wp-content/uploads/2018/01/Screen-Shot-2018-01-31-at-15.37.49.png)](https://www.robustperception.io/wp-content/uploads/2018/01/Screen-Shot-2018-01-31-at-15.37.49.png)

_Interested in gaining more operational insights with Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by Conor Broderick in [Posts](https://www.robustperception.io/category/posts)
