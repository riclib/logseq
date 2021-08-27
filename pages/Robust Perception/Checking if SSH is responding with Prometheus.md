---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/checking-if-ssh-is-responding-with-prometheus
author: [[Brian Brazil]] 
---
# Checking if SSH is responding with Prometheus

> The blackbox_exporter allows for a variety of network checks to be performed, with many common modules available out of the box.

---
The [blackbox_exporter](https://github.com/prometheus/blackbox_exporter) allows for a variety of network checks to be performed, with many common modules available out of the box.

[Prometheus](https://prometheus.io/) is a whitebox monitoring system, ingesting metrics exposed from inside applications. Sometimes though you want to check how things look from the outside, which is to say blackbox monitoring. For this Prometheus offers the [blackbox_exporter](https://github.com/prometheus/blackbox_exporter) . As an example let's check for the SSH returning the banner.

First we run the blackbox exporter:
```shell
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.10.0/blackbox_exporter-0.10.0.linux-amd64.tar.gz
tar -xzf blackbox_exporter-*.linux-amd64.tar.gz
cd blackbox_exporter-*
./blackbox_exporter
```
If you visit [http://localhost:9115/probe?target=127.0.0.1:22&module=ssh_banner](http://localhost:9115/probe?target=127.0.0.1:22&module=ssh_banner)  it'll test to see if SSH on the local machine is responding.

Now let's get Prometheus to use this:
```shell
wget https://github.com/prometheus/prometheus/releases/download/v2.0.0/prometheus-2.0.0.linux-amd64.tar.gz
tar -xzf prometheus-*.tar.gz
cd prometheus-*
cat <<'EOF' > prometheus.yml
global:
  scrape_interval: 10s
scrape_configs:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [ssh_banner]
    static_configs:
      - targets:
        - 127.0.0.1   # Targets to probe
    relabel_configs:
      # Ensure port is 22, pass as URL parameter
      - source_labels: [__address__]
        regex: (.*?)(:.*)?
        replacement: ${1}:22
        target_label: __param_target
      # Make instance label the target
      - source_labels: [__param_target]
        target_label: instance
      # Actually talk to the blackbox exporter though
      - target_label: __address__
        replacement: 127.0.0.1:9115
EOF
./prometheus
```
You can now see the result of `probe_success` in the [expression browser](http://localhost:9090/graph?g0.range_input=1h&g0.expr=probe_success&g0.tab=1)!

While this example only checks the local machine, you could get a list of targets from any service discovery method - for example [EC2](https://www.robustperception.io/automatically-monitoring-ec2-instances/) or [Consul](https://www.robustperception.io/finding-consul-services-to-monitor-with-prometheus/) instead of just `static_configs`.

The blackbox exporter includes some useful modules out of the box, such as HTTP, TCP, POP3S, IRC and ICMP. The config in `blackbox.yml` can be expanded to add additional modules that cater to your needs.

One nifty feature is that if a module ends up using TLS/SSL, the exporter will automatically expose when the cert chain will expire. This makes it easy to alert on [soon to expire SSL certs](https://www.robustperception.io/get-alerted-before-your-ssl-certificates-expire/).
