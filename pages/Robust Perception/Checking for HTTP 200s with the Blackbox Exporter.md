---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/checking-for-http-200s-with-the-blackbox-exporter
author: [[Brian Brazil]] 
---
> It's easy to check if HTTP and HTTPS endpoints are working with the Blackbox Exporter.

# Checking for HTTP 200s with the Blackbox Exporter


It's easy to check if HTTP and HTTPS endpoints are working with the Blackbox Exporter.

The Blackbox exporter supports several different types of probes, which includes HTTP. To demonstrate this let's start by downloading and running the blackbox exporter:

wget https://github.com/prometheus/blackbox\_exporter/releases/download/v0.12.0/blackbox\_exporter-0.12.0.linux-amd64.tar.gz
tar -xzf blackbox\_exporter-\*.linux-amd64.tar.gz
cd blackbox\_exporter-\*
./blackbox\_exporter

How the Blackbox exporter works is that the `/probe` endpoint takes `module` and `target` URL parameters. Modules are configured in `blackbox.yml` and the default config includes a `http_2xx` module which does a HTTP probe which considers any 2xx HTTP response successful. So if you visit [http://localhost:9115/probe?module=http\_2xx&target=https://www.robustperception.io/](http://localhost:9115/probe?module=http_2xx&target=https://www.robustperception.io) you will see the result of a probe of [https://www.robustperception.io/](https://www.robustperception.io/). In particular look at the `probe_success` metric, which will be 1 if the probe succeeded and 0 if it failed.

Now that the exporter is working, let's setup a Prometheus to use it:

wget https://github.com/prometheus/prometheus/releases/download/v2.4.2/prometheus-2.4.2.linux-amd64.tar.gz
tar -xzf prometheus-\*.tar.gz
cd prometheus-\*
cat <<'EOF' > prometheus.yml
global:
 scrape\_interval: 10s
scrape\_configs:
 - job\_name: blackbox
   metrics\_path: /probe
   params:
     module: \[http\_2xx\]
   static\_configs:
    - targets:
       - https://www.robustperception.io/probe
       - http://prometheus.io/blog
   relabel\_configs:
    - source\_labels: \[\_\_address\_\_\]
      target\_label: \_\_param\_target
    - source\_labels: \[\_\_param\_target\]
      target\_label: instance
    - target\_label: \_\_address\_\_
      replacement: 127.0.0.1:9115 # The blackbox exporter.
EOF
./prometheus

The `relabel_configs` change the usual targets into URL parameters on the blackbox exporter. As you can see, paths can be included and HTTP and HTTPS are handled in the same way.

If you wait a few seconds, you will see the result of `probe_success` in the [expression browser.](http://localhost:9090/graph?g0.range_input=1h&g0.expr=probe_success&g0.tab=1) You may see a surprising failure if you don't have a working IPv6 setup, as the Blackbox exporter will prefer an IPv6 address if one is returned by DNS. You can adjust this behaviour by adding `preferred_ip_protocol: "ip4"` to the module's configuration.

If you wanted to alert on probes failing you should look at both the `up` and `probe_success` metrics, to catch either the exporter or target having issues:

groups:
- name: example
  rules:
   - alert: ProbeFailing
     expr: up{job="blackbox"} == 0 or probe\_success{job="blackbox"} == 0
     for: 10m

_Want to know more about blackbox monitoring? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
