---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/using-json-file-service-discovery-with-prometheus
author: [[Brian Brazil]] 
---
# Using JSON file service discovery with Prometheus

> Prometheus offers a number of ways to find the targets to scrape, DNS, EC2, Consul, Kubernetes, Zookeeper and Marathon. But what if what you aren't using one of those?

---
Prometheus offers a number of ways to find the targets to scrape, DNS, EC2, Consul, Kubernetes, Zookeeper and Marathon. But what if what you aren't using one of those?

It's not possible for [Prometheus](https://prometheus.io/) to support every possible environment, and attempting to do so out of the box would make things rather unwieldy. Instead in a number of places we offer ways for you to hook in and provide the functionality you need. One of those is service discovery, where file-based discovery lets you have your configuration management, a cronjob or whatever else you'd like produce files that Prometheus reads targets from.

An example setup for a MySQL master with two slaves in a production environment could look like this:

```json
[
  {
    "targets": [ "myslave1:9104", "myslave2:9104" ],
    "labels": {
      "env": "prod",
      "job": "mysql_slave"
    }
  },
  {
    "targets": [ "mymaster:9104" ],
    "labels": {
      "env": "prod",
      "job": "mysql_master"
    }
  },
  {
    "targets": [ "mymaster:9100", "myslave1:9100", "myslave2:9100" ],
    "labels": {
      "env": "prod",
      "job": "node"
    }
  }
]
```
If the above was in a file called `targets.json`, then you could use the following Prometheus configuration:

```yaml
scrape_configs:
  - job_name: 'dummy'  # This is a default value, it is mandatory.
    file_sd_configs:
      - files:
        - targets.json
```
Every time the file changes Prometheus will automatically reread the file, no need for a restart or [reload](http://www.robustperception.io/reloading-prometheus-configuration/). You can also use globs to select multiple files in a directory too. If you'd like more information, see the [project documentation](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#file_sd_config).
