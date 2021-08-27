---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/get-alerted-before-your-ssl-certificates-expire
author: [[Brian Brazil]] 
---
# Get alerted before your SSL certificates expire

> The most common way to learn about the expiry date of your website's SSL certificate is after it has expired. The blackbox exporter combined with Prometheus can let you know well in advance, letting you renew your certificate before users complain.

---
The most common way to learn about the expiry date of your website's SSL certificate is after it has expired. The [blackbox exporter](https://github.com/prometheus/blackbox_exporter) combined with Prometheus can let you know well in advance, letting you renew your certificate before users complain.

To start with, download, compile and run the blackbox exporter:
```shell
git clone git@github.com:prometheus/blackbox_exporter.git
cd blackbox_exporter
make
./blackbox_exporter
```
If you visit [http://localhost:9115/probe?target=https://example.com&module=http_2xx](http://localhost:9115/probe?target=https://example.com&module=http_2xx) the blackbox exporter will probe [https://example.com](https://example.com/) and report several metrics. One of them is `probe_ssl_earliest_cert_expiry` which is the time the  certificate chain will no longer be valid. This is automatically reported for any SSL endpoints.

The next step is to hook this in to Prometheus, and create an alert. We'll usually want to probe multiple endpoints coming from service discovery with the same blackbox exporter, so we use relabelling to convert the target addresses to URL parameters:

```shell
wget https://github.com/prometheus/prometheus/releases/download/v2.4.3/prometheus-1.4.1.linux-amd64.tar.gz
tar -xzf prometheus-*.tar.gz
cd prometheus-*
cat << 'EOF' > prometheus.yml
rule_files:
  - ssl_expiry.rules
scrape_configs:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - example.com  # Target to probe
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115  # Blackbox exporter.
        EOF 
cat << 'EOF' > ssl_expiry.rules 
groups: 
  - name: ssl_expiry.rules 
    rules: 
      - alert: SSLCertExpiringSoon 
        expr: probe_ssl_earliest_cert_expiry{job="blackbox"} - time() < 86400 * 30 
        for: 10m
EOF
./prometheus
```
If you visit [http://localhost:9090/alerts](http://localhost:9090/alerts) you'll see your new alert, ready to let you know you 30 days before your certs expire!
