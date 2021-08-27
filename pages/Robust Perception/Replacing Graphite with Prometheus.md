---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/replacing-graphite-with-prometheus
author: [[Brian Brazil]] 
---
# Replacing Graphite with Prometheus

> Many of the companies I talk to either want to move off Graphite, or are already doing so. Let's look at how to get your existing data that's going to Graphite into Prometheus instead.

---
Many of the companies I talk to either want to move off Graphite, or are already doing so. Let's look at how to get your existing data that's going to Graphite into [Prometheus](https://prometheus.io/) instead.

Prometheus uses an advanced pull model to collect metrics, allowing for easy detection of failed servers and making it easy to run your own development Prometheus server for against production data. As Graphite uses a push model, we'll use the [Graphite Exporter](https://github.com/prometheus/graphite_exporter) to bridge this gap.

[![Replacing Graphite with Prometheus](http://www.robustperception.io/wp-content/uploads/2015/11/Replacing-Graphite-with-Prometheus.png)](http://www.robustperception.io/wp-content/uploads/2015/11/Replacing-Graphite-with-Prometheus.png)

First let's get the Graphite Exporter running:
```shell
wget https://github.com/prometheus/graphite_exporter/releases/download/v0.2.0/graphite_exporter-0.2.0.linux-amd64.tar.gz
tar -xzf graphite_exporter-0.2.0.linux-amd64.tar.gz
cd graphite_exporter-*
./graphite_exporter &
```
Now change your existing systems to send their samples to the Graphite Exporter on port 9109. It accepts the [Graphite plaintext protocol](http://graphite.readthedocs.org/en/latest/feeding-carbon.html#the-plaintext-protocol) on both TCP and UDP. Once that's done, you'll see your metrics on [http://localhost:9108/metrics](http://localhost:9108/metrics).

Next let's setup a quick Prometheus server:
```shell
wget https://github.com/prometheus/prometheus/releases/download/v2.0.0/prometheus-2.0.0.linux-amd64.tar.gz
tar -xzf prometheus-2.0.0.linux-amd64.tar.gz
cd prometheus-*
cat <<'EOF' > prometheus.yml
global:
 scrape_interval: 10s
 evaluation_interval: 10s
scrape_configs:
 - job_name: 'graphite'
   honor_labels: true
   static_configs:
    - targets:
      - localhost:9108
EOF
./prometheus
```
If you visit [http://localhost:9090](http://localhost:9090/) you'll be able to play with your new Prometheus server and take advantage of the power of PromQL. Beyond the expression browser, you can also [setup Grafana](http://www.robustperception.io/setting-up-grafana-for-prometheus/) for dashboards, create [alerting](http://www.robustperception.io/alerting-on-down-instances/) rules and use [mapping configurations](https://github.com/prometheus/graphite_exporter#metric-mapping-and-configuration) to add labels to get the most out of Prometheus!
