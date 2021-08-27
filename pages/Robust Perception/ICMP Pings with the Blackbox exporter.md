---
created: [[Aug 4th, 2021]]
type: #clipping
status: #reviewed
tags: prometheus
source: https://www.robustperception.io/icmp-pings-with-the-blackbox-exporter
author: [[Brian Brazil]]
---

> The Blackbox exporter can perform ICMP probes. Let's see how.

# ICMP Pings with the Blackbox exporter

The Blackbox exporter can perform ICMP probes. Let's see how.

First download and run the blackbox_exporter:

```shell
wget https://github.com/prometheus/blackbox\_exporter/releases/download/v0.12.0/blackbox\_exporter-0.12.0.linux-amd64.tar.gz
tar -xzf blackbox\_exporter-\*.linux-amd64.tar.gz
cd blackbox\_exporter-\*
sudo ./blackbox\_exporter
```

`sudo` is used here as ICMP typically requires raw socket access, and running as root is one way to provide the necessary privileges.

If you visit [http://localhost:9115/probe?module=icmp&target=localhost](http://localhost:9115/probe?module=icmp&target=localhost) the probe will be performed, and should include `probe_success 1` indicating that it works. You can replace the target with any other hostname that you wish to probe.

Prometheus can use the Blackbox exporter to do ICMP probes against many targets with a little relabelling in your `prometheus.yml`:

``` yaml
scrape_configs:
	- job_name: 'blackbox'
 metrics_path: /probe
 params:
   module: [icmp]
 static_configs:
- targets:
	- localhost
	- prometheus.io
	- robustperception.io
	      relabel_configs:
- source\_labels: [__address__]
  target\_label: __param_target
- source\_labels: [__param_target]
  target\_label: instance
- target\_label: __address__
  replacement: 127.0.0.1:9115 \# This is your blackbox exporter.
```

Here the usual target address become URL parameters to your blackbox exporter.

If you don't have a working IPv6 setup you may find that some targets can't be probed as the blackbox exporter prefers using an IPv6 address if there is one available. If you add &debug=true to the URL there'll be an error message like "sendto: cannot assign requested address" in this case. To handle this you can create a module that prefers IPv4 in your `blackbox.yml`:

``` yaml
modules:
icmp\_ipv4:
 prober: icmp
 icmp:
 preferred\_ip\_protocol: ip4
```

And then use the `icmp_ipv4` module.