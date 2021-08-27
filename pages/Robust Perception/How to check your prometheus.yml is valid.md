---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/how-to-check-your-prometheus-yml-is-valid
author: [[Brian Brazil]] 
---
> It's nice to check that your configuration is valid before pushing to production.

# How to check your prometheus.yml is valid


It's nice to check that your configuration is valid before pushing to production.

Prometheus will gracefully fail to reload if there's a bad configuration, but will fail to start if there isn't one at startup. Thus it's wise to check that the configuration is good before checking it in via continuous integration or similar mechanisms.

To facilitate this, `promtool` which comes with Prometheus has a `check config` command. Let's download Prometheus and create an invalid config:

wget https://github.com/prometheus/prometheus/releases/download/v2.5.0/prometheus-2.5.0.linux-amd64.tar.gz
tar -xzf prometheus-\*.tar.gz
cd prometheus-\*
cat >prometheus.yml <<'EOF'
scrape\_configs:
- job\_name: prometehus
static\_configs:
  - targets: \['localhost:9090'\]
EOF

If you now check the configuration, you'll see an error:

./promtool check config prometheus.yml
Checking prometheus.yml
  FAILED: parsing YAML file prometheus.yml: yaml: unmarshal errors:
  line 3: field static\_configs not found in type config.plain

and the exit code will be 1, indicating failure.

If you fix the last two lines to be correctly indented and check the config again it will now pass:

./promtool check config prometheus.yml 
Checking prometheus.yml
 SUCCESS: 0 rule files found

and the exit code will be 0.

This sort of feature isn't limited to Prometheus, the Alertmanager's `amtool` has a similar feature too.

_Prometheus configuration got you down? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
