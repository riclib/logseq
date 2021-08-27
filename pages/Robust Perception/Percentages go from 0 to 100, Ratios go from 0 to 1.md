---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/percentages-go-from-0-to-100-ratios-go-from-0-to-1
author: [[Brian Brazil]] 
---
# Percentages go from 0 to 100, Ratios go from 0 to 1

> While doing research for implementing exporters, I've noticed some confusion around ratios and percentages that I'd like to clear up.

---
While doing research for implementing exporters, I've noticed some confusion around ratios and percentages that I'd like to clear up.

The [master/cpu_percent](http://mesos.readthedocs.io/en/0.24.1/monitoring/#master-nodes) metric in Mesos is an example of this. As it has the word "percent" in it, you'd expect it to have values from 0 to 100 indicating the current cpu usage percentage.

In fact it has values going from 0 to 1. So it's not a percentage at all, it's a ratio!

I've seen this mistake in both directions. Ratios and percentages are not interchangeable terms, one is one hundred times the other. When you're writing an exporter check that the value of any metrics called ratio or percentage metrics actually match the name.

As the Prometheus convention is to use base units, exposing a ratio is preferred. Even better is if you can expose the numerator and denominator as separate metrics, and leave the ratio calculations to Prometheus.

(For the nitpickers: Yes yes, ratios can have values outside 0..1 and percentages values outside 0..100).
