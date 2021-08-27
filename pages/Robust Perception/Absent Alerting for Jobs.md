---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus
source: https://www.robustperception.io/absent-alerting-for-jobs
author: [[Brian Brazil]]
---

> Alerting on numbers being too big or small is easy with Prometheus. But what if the numbers go missing?

# Absent Alerting for Jobs

Alerting on numbers being too big or small is easy with Prometheus. But what if the numbers go missing?

In normal operations your Prometheus discovers your targets, scrapes them, and will run any alerting rules you have defined against them. But that can go wrong. Your instances might disappear from service discovery for example, which would result in any alerts such as `avg by (job)(up) < 0.5` returning nothing rather than alerting. As _previously discussed_[^1] when there's no input, aggregators produce no output.

Accordingly it is advised to have alerts on all the targets for a job going away, for example:

```yaml
groups:
- name: example
rules:
	- alert: MyJobMissing
 expr: absent(up{job="myjob"})
 for: 10m
```

This uses the `absent` function. If the given selector doesn't match anything then a single time series with the value 1 and the labels of any equality matchers is returned. In this case for example the alert would have a label of `job="myjob"`. If there are matched series, then nothing is returned.

Prometheus can't know which sets of labels are meant to exist, so you'd need one such alert for each of your jobs. This only applies to `absent` and other cases where time series are missing though, if you just want to detect if a target is down you can do so with _one alerting rule_[^2] as usual.

[^1] [[Why can count(x  5) not return 0]] 
[^2] [[Alerting on gauges in Prometheus 2/0]]