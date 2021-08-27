---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/configuring-blackbox-exporter-timeouts
author: [[Brian Brazil]] 
---
# Configuring Blackbox exporter timeouts

> A blog on monitoring, scale and operational Sanity

---
A blog on monitoring, scale and operational Sanity

Wondering how the cool kids are configuring their Blackbox probe timeouts these days?

With the latest [0.7.0 release](https://github.com/prometheus/blackbox_exporter/releases/tag/v0.7.0) of the Blackbox exporter comes a new way to configure timeouts for probes.  
Previously, timeouts for probes would be set individually via configuration of the Blackbox exporter itself.  
Now the timeout for probes may be set in Prometheus.

In practice this works by configuring [`scrape_timeout` in the `scrape_config`](https://prometheus.io/docs/operating/configuration/#%3Cscrape_config%3E) to automatically determine the probe timeout. Note that it is slightly reduced to allow for network delays.

This can be further limited by the `timeout` in the Blackbox exporter [config file](https://github.com/prometheus/blackbox_exporter/blob/master/CONFIGURATION.md#module) of the individual probe. If neither is specified, it defaults to 10 seconds.

Under the covers this works via the `X-Prometheus-Scrape-Timeout-Seconds` HTTP header which Prometheus sends with each scrape.
