---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/productive-prometheus-python-parsing
author: [[Brian Brazil]] 
---
# Productive Prometheus Python Parsing

> Prometheus client libraries don't just export metrics in our format, they can parse that format too.

---
Prometheus client libraries don't just export metrics in our format, they can parse that format too.

Even if you don't run Prometheus, the Prometheus exposition format can be useful to you. By having a standard format exposed by a [wide variety](https://prometheus.io/docs/instrumenting/exporters/) of integrations, you gain access to metrics that you'd have to otherwise figure out how to extract yourself. You might use this as part of an auto-scaling system, or even to send the metrics on to another monitoring system like [Graphite](https://www.robustperception.io/exporting-to-graphite-with-the-prometheus-python-client/).

Let's look at an example where we pull down metrics from a node exporter.

First install the python client and [requests](http://docs.python-requests.org/en/master/), if you don't already have them:

$ pip install prometheus_client requests

Then we can fetch the data we need, and print it:

```python
from prometheus_client.parser import text_string_to_metric_families
import requests

metrics = requests.get("http://demo.robustperception.io:9100/metrics").content

for family in text_string_to_metric_families(metrics):
  for sample in family.samples:
    print("Name: {0} Labels: {1} Value: {2}".format(*sample))
```
These few lines of code can be extended to do whatever you like!
