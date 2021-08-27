---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/alerting-on-down-instances
author: [[Brian Brazil]] 
---
# Alerting on Down Instances

> Knowing which instances of your services and which machines in your fleet are no longer responding is a common requirement. Whether it's to get someone to investigate or to drive automation, in this post I'll look at how you can do it with Prometheus.

---
Knowing which instances of your services and which machines in your fleet are no longer responding is a common requirement. Whether it's to get someone to investigate or to drive automation, in this post I'll look at how you can do it with [Prometheus](https://prometheus.io/).

I'll presume you've already setup monitoring and are scraping the instances, whether that be the [Node Exporter](https://github.com/prometheus/node_exporter) for machine monitoring or some other exporter. To generate an alert for each instance that has been down for 10 minutes:
```yaml
groups:
- name: node.rules
  rules:
  - alert: InstanceDown
    expr: up{job="node"} == 0
    for: 10m
```

The power of labels means that you only need to define this alert once, and it automatically applies to all of your instance with a `node` label!

A single instance going down shouldn't be worth waking someone up over. How about only alerting when 25% of the instances are down?
```yaml
groups:
- name: node.rules  
  rules:
  - alert: InstancesDown
    expr: avg(up{job="node"}) BY (job) < 0.75
    for: 10m
```
These simple examples show just a small glimpse of the power of Prometheus alerting.
