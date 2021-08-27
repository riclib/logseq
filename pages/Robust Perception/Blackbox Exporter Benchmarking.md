---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/blackbox-exporter-benchmarking
author: [[Brian Brazil]] 
---
# Blackbox Exporter Benchmarking

> Have you ever wondered how many CPU seconds it takes to probe an instance via TCP or HTTP 100, 1,000, or 10,000 times?

---
Have you ever wondered how many CPU seconds it takes to probe an instance via TCP or HTTP 100, 1,000, or 10,000 times?

In this blogpost we'll be taking a look at how many CPU seconds in total are used across four cores when probing an external service using the [blackbox exporter](http://github.com/prometheus/blackbox_exporter).

In case you are unfamiliar with the concept, blackbox monitoring is the monitoring of a service whereby no participation of the monitored system is required. The monitoring only observes external functionality; essentially what the outside world sees.

The blackbox exporter provided by the [Prometheus organisation](http://prometheus.io/) allows for blackbox probing of endpoints over HTTP, HTTPS, DNS, TCP, and ICMP.

For this test I used the [Intel Core i7-4980HQ](https://ark.intel.com/products/83503/Intel-Core-i7-4980HQ-Processor-6M-Cache-up-to-4_00-GHz) CPU.

The following results were recorded when calculating the average time it took for the blackbox exporter to send X amount of TCP/HTTP probes to a locally running Prometheus instance:
```
TCP Probes:
No. of probes
100
1,000
10,000

Time (s)
0.07s
0.81s
7.85s

HTTP Probes:
No. of probes

100
1,000
10,000

Time (s)
0.09s
1.01s
9.6s
```

As we can see, it is possible to do 1,000 probes per second with relatively little cost in regards to CPU usage.

The results of these probes can be scraped by a Prometheus instance, queried, graphed, or turned into alerts.
