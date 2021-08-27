---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/monitoring-mysqld-with-prometheus
author: [[Brian Brazil]] 
---
# Monitoring MySQLd with Prometheus

> The Prometheus ecosystem contains a multitude of integrations, both officially supported and third party. Let's have a look at how to use the mysqld_exporter.

---
The Prometheus ecosystem contains a [multitude of integrations](https://prometheus.io/docs/instrumenting/exporters/), both officially supported and third party. Let's have a look at how to use the mysqld_exporter.

MySQL is a popular database system, which exposes a wide variety of metrics but not in a format Prometheus can directly consume. The [mysqld_exporter](https://github.com/prometheus/mysqld_exporter) bridges this gap.

I'm going to assume you already have a working MySQLd installed. We first need to create a user that can connect locally with appropriate permissions:
```sql
CREATE USER 'mysqld_exporter'@'localhost' IDENTIFIED BY 'a_password';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'mysqld_exporter'@'localhost'
  WITH MAX_USER_CONNECTIONS 3;
```

Then download and run the mysqld_exporter:

```shell
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.9.0/mysqld_exporter-0.9.0.linux-amd64.tar.gz
tar -xzf mysqld_exporter-0.9.0.linux-amd64.tar.gz
export DATA_SOURCE_NAME='mysqld_exporter:a_password@unix(/var/run/mysqld/mysqld.sock)/'
./mysqld_exporter
```
You may need to adjust the location of the Unix socket.

If you visit [http://localhost:9104/metrics](http://localhost:9104/metrics) you'll see your metrics!

Let's setup a Prometheus to scrape the exporter:
```shell
wget https://github.com/prometheus/prometheus/releases/download/v2.0.0/prometheus-2.0.0.linux-amd64.tar.gz
tar -xzf prometheus-2.0.0.linux-amd64.tar.gz
cd prometheus-*
cat <<'EOF' > prometheus.yml
global:
 scrape_interval: 10s
 evaluation_interval: 10s
scrape_configs:
 - job_name: 'mysqld'
   static_configs:
    - targets:
      - localhost:9104
EOF
./prometheus &
```
From there you can go to [http://localhost:9090/graph](http://localhost:9090/graph) and plot expressions such as `irate(mysql_global_status_queries[1m])` to get the query rate.

The mysqld_exporter can provide a wide wealth of information, however for performance reasons most of this is disabled by default. The [official docs](https://github.com/prometheus/mysqld_exporter#collector-flags) have the flags you can use to enable more metrics.
