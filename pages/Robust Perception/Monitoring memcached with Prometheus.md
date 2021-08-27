---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/monitoring-memcached-with-prometheus
author: [[Brian Brazil]] 
---
> As with many applications, there's an exporter for memcached.

# Monitoring memcached with Prometheus


As with many applications, there's an exporter for [memcached](https://memcached.org/).

To demonstrate, let's get a memcached up and running using Docker so you don't have to compile it by hand:

docker run -p11211:11211 -d memcached:1.5.16

You can see the stats it provides with:

echo stats | nc localhost 11211

To get this into a format usable by Prometheus, you can use the [memcached\_exporter](https://github.com/prometheus/memcached_exporter):

wget https://github.com/prometheus/memcached\_exporter/releases/download/v0.5.0/memcached\_exporter-0.5.0.linux-amd64.tar.gz
tar -xzf memcached\_exporter-0.5.0.linux-amd64.tar.gz
cd memcached\_exporter-0.5.0.linux-amd64/
./memcached\_exporter

Which will expose metrics on [http://localhost:9150/metrics](http://localhost:9150/metrics). Now you can run a Prometheus to scrape this:

wget https://github.com/prometheus/prometheus/releases/download/v2.10.0/prometheus-2.10.0.linux-amd64.tar.gz
tar -xzf prometheus-2.10.0.linux-amd64.tar.gz
cd prometheus-\*
cat <<'EOF' > prometheus.yml
global:
 scrape\_interval: 10s
 evaluation\_interval: 10s
scrape\_configs:
 - job\_name: 'memcached'
   static\_configs:
    - targets:
      - localhost:9150
EOF
./prometheus &

From there you can go to [http://localhost:9090/graph](http://localhost:9090/graph) and plot expressions such as `rate(memcached_commands_total[1m])` to get the query rate.

_Have questions about monitoring applications with Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
