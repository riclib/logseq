---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/relabel_configs-vs-metric_relabel_configs
author: [[Brian Brazil]] 
---

# relabel_configs vs metric_relabel_configs

> We've looked at the full Life of a Label. Let's focus on one of the most common confusions around relabelling.

---
We've looked at the full [Life of a Label](https://www.robustperception.io/life-of-a-label/). Let's focus on one of the most common confusions around relabelling.

It's not uncommon for a user to share a Prometheus config with a valid `relabel_configs` and wonder why it isn't taking effect. This is often resolved by using `metric_relabel_configs` instead (the reverse has also happened, but it's far less common). So let's shine some light on these two configuration options.

Prometheus needs to know what to scrape, and that's where service discovery and `relabel_configs` come in. Relabel configs allow you to select [which targets you want scraped](https://www.robustperception.io/automatically-monitoring-ec2-instances/)[^1], and [what the target labels will be](https://www.robustperception.io/finding-consul-services-to-monitor-with-prometheus/)[^2]. So if you want to say scrape this type of machine but not that one, use `relabel_configs`.

`metric_relabel_configs` by contrast are applied after the scrape has happened, but before the data is ingested by the storage system. So if there are some [expensive metrics you want to drop](https://www.robustperception.io/dropping-metrics-at-scrape-time-with-prometheus/), or labels coming from the scrape itself (e.g. from the /metrics page) that you want to manipulate that's where  `metric_relabel_configs` applies.

So as a simple rule of thumb: `relabel_config` happens before the scrape, `metric_relabel_configs` happens after the scrape. And if one doesn't work you can always try the other!
- [^1] [[Automatically monitoring EC2 Instances]] 
  [^2] [[Finding Consul services to monitor with Prometheus]]