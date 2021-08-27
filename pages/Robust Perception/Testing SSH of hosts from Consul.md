---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/testing-ssh-of-hosts-from-consul
author: [[Brian Brazil]] 
---
> We've previously looked at scraping services from consul and ssh checks. How can we combine those?

# Testing SSH of hosts from Consul


We've previously looked at [scraping services from consul](https://www.robustperception.io/finding-consul-services-to-monitor-with-prometheus) and [ssh checks](https://www.robustperception.io/checking-if-ssh-is-responding-with-prometheus). How can we combine those?

The use of relabelling for the blackbox exporter isn't tied to `static_configs`, you can use it with any service discovery mechanism. I'll presume you already have a working Consul setup.

To demonstrate first we run the blackbox exporter:

wget https://github.com/prometheus/blackbox\_exporter/releases/download/v0.16.0/blackbox\_exporter-0.16.0.linux-amd64.tar.gz
tar -xzf blackbox\_exporter-\*.linux-amd64.tar.gz
cd blackbox\_exporter-\*amd64
./blackbox\_exporter

Then run Prometheus:

wget https://github.com/prometheus/prometheus/releases/download/v2.15.2/prometheus-2.15.2.linux-amd64.tar.gz
tar -xzf prometheus-\*.tar.gz
cd prometheus-\*amd64
cat <<'EOF' > prometheus.yml
global:
  scrape\_interval: 10s
scrape\_configs:
  - job\_name: 'blackbox'
    metrics\_path: /probe
    params:
      module: \[ssh\_banner\]
    # Use consul as SD.
    consul\_sd\_configs:
      - server: 'localhost:8500'
    relabel\_configs:
      - source\_labels: \[\_\_address\_\_\]
        regex: (.\*?)(:.\*)?
        replacement: ${1}:22
        target\_label: \_\_param\_target
      - source\_labels: \[\_\_param\_target\]
        target\_label: instance
      - target\_label: \_\_address\_\_
        replacement: 127.0.0.1:9115
EOF
./prometheus

You'll note that this is very similar to the previous [ssh example](https://www.robustperception.io/checking-if-ssh-is-responding-with-prometheus), which is to be expected. We are cheating a little here and as we're taking advantage of the fact that if relabelling for a given scrape config produces identical targets, those targets are merged. So even if a host has multiple Consul services (which it usually will), we'll end up with only one target per host in Prometheus which is fine for ssh running on a fixed well known port.

The Consul service discovery provides some metadata, so if you were probing something other than ssh on port 22 you could for example limit it to hosts that run the `consul` service:

      - source\_labels: \[\_\_meta\_consul\_service\]
        regex: consul
        action: keep

Or add target labels based on tags, probe specific ports for a given service and so on.

This shows some of the power of relabelling. There's no special code or configuration needed to tie Consul to the blackbox exporter, instead you can compose existing features.

_Wondering about service discovery? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
