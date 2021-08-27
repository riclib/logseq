---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/which-targets-have-the-most-samples
author: [[Brian Brazil]] 
---
# Which targets have the most samples?

> We previously looked at finding your biggest metrics, that involves an expensive query though. A new feature in Prometheus 1.3 offers another approach.

---
We [previously looked](https://www.robustperception.io/which-are-my-biggest-metrics/) at finding your biggest metrics, that involves an expensive query though. A new feature in Prometheus 1.3 offers another approach.

`scrape_samples_scraped`, like `[up](https://www.robustperception.io/whats-up-doc/)`, is a time series automatically generated by Prometheus as part of doing a scrape. It reports the numbers of samples that the target produced, before `metric_relabel_configs` is applied.

This can be used as a very quick and cheap way to pinpoint a target that has started to produce many more samples than it used to. To do so is as simple as putting the expression `scrape_samples_scraped` in the graph view in the expression browser:

[![](https://www.robustperception.io/wp-content/uploads/2016/12/Screenshot_2016-12-07_19-08-06.png)](https://www.robustperception.io/wp-content/uploads/2016/12/Screenshot_2016-12-07_19-08-06.png)

In this case the recent load increase is obviously due to SNMP, so you know where to start debugging.

A related metric is `scrape_samples_post_metric_relabeling`, which was added in 1.5.0.
