---
created: [[Aug 4th, 2021]]
type: #clipping
tags: #prometheus
source: https://www.robustperception.io/how-to-have-labels-for-machine-roles
author: [[Brian Brazil]]
---
> It's a best practice with Prometheus that target labels should be constant over a target's entire lifetime. On the other hand it's useful to aggregate metrics across all the machines that are currently Apache servers. How can we do that?

# How to Have Labels for machine Roles

It's a best practice with [Prometheus](https://prometheus.io/) that target labels should be constant over a target's entire lifetime. On the other hand it's useful to aggregate metrics across all the machines that are currently Apache servers. How can we do that?

A key concept in Prometheus is that you want continuity in your time series, that is that they don't change labels and become a different time series. Something like a Chef role which may change as the role of a machine changes isn't a good target label. Every time the roles changed there'd be discontinuities in the graphs and alerts would get reset.

That means though that you can't easily aggregate metrics like CPU usage across machines in a role. The good news is that the textfile collector and grouping modifiers offer a way to do this.

We'll start from scratch, download and run the node exporter with the textfile collector:
``` shell
wget https://github.com/prometheus/node\_exporter/releases/download/v0.15.1/node\_exporter-0.15.1.linux-amd64.tar.gz
tar -xzf node\_exporter-0.15.1.linux-amd64.tar.gz
cd node\_exporter-\*
mkdir textfile\_collector
./node\_exporter -collector.textfile.directory textfile\_collector &
```

Let's say that this machine runs postfix and apache. We'll add a metric with these roles:
``` shell
cat <<EOF > textfile\_collector/roles.prom
machine\_role{role="postfix"} 1
machine\_role{role="apache"} 1
EOF
```

This would usually be done by your configuration management system. If you visit [http://localhost:9100/metrics](http://localhost:9100/metrics) you'll see your new metrics.

Next we'll quickly setup a Prometheus server to scrape this:
``` shell
wget https://github.com/prometheus/prometheus/releases/download/v2.0.0/prometheus-2.0.0.linux-amd64.tar.gz
tar -xzf prometheus-2.0.0.linux-amd64.tar.gz
cd prometheus-\*
cat <<'EOF' > prometheus.yml
global:
scrape\_interval: 10s
evaluation\_interval: 10s
scrape\_configs:
- job\_name: 'node'
  static\_configs:
    - targets:
      - localhost:9100
EOF
./prometheus
```

If we wanted metrics for apache role with the role label attached, you can use group\_left in the [expression browser](http://localhost:9090/graph):
``` c
up \* on (instance, job) group\_left(role) machine\_role{role="apache"}
```
This can then be aggregated as normal:
``` c
sum by (job, role)(
  up \* on (instance, job) group\_left(role) machine\_role{role="apache"}
)
```

This technique lets you have the benefits of attaching labels to your targets, without the downsides of having labels changing over the lifetime of the target. A similar approach works for applications too![^1]
- [^1] [[Exposing the software version to Prometheus]]