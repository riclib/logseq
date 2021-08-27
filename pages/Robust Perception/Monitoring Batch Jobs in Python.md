---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/monitoring-batch-jobs-in-python
author: [[Brian Brazil]] 
---
# Monitoring Batch Jobs in Python

> Prometheus monitoring is usually against on long-lived daemons, but what if you've a batch job that you want to monitor?

---
Prometheus monitoring is usually against on long-lived daemons, but what if you've a batch job that you want to monitor?

When monitoring batch jobs, such as cronjobs, the main thing you care about is when it last succeeded. For example if you've a cronjob that runs every hour and needs to work at least once every few hours, then you want to alert if it hasn't worked for at least two runs - rather than on every individual failure. It's also useful to track how long batch jobs take over time.

[Prometheus](https://prometheus.io/) is primarily a pull-based monitoring system. Batch jobs should use the Pushgateway, which will persist metrics for them

First install the [Prometheus Python client](https://github.com/prometheus/client_python):

pip install prometheus_client

Then in your batch job push metrics to the Pushgateway:
```python
from prometheus_client import Gauge,CollectorRegistry,pushadd_to_gateway

registry = CollectorRegistry()
duration = Gauge('mybatchjob_duration_seconds',
    'Duration of batch job', registry=registry)
try:
  with duration.time():
    pass  # Your code here
except: pass
else:
  last_success = Gauge('mybatchjob_last_success', 
      'Unixtime my batch job last succeeded', registry=registry)
  last_success.set_to_current_time()
finally:
  pushadd_to_gateway('localhost:9091', job='my_batch_job', registry=registry)
```
The `mybatchjob_last_success` metric is only pushed when we succeed. As we're using `pushadd_to_gateway` rather than `push_to_gateway` a failed run won't overwrite the value of a previous success. `mybatchjob_duration_seconds` is always pushed, so you can graph it over time.

Once you've setup Prometheus to scrape the Pushgateway, you can add an alert in a rule file:
```yaml
groups:
- name: test.rules
  rules:
  - alert: MyBatchJobNoRecentSuccess
    expr: time() - mybatchjob_last_success{job="my_batch_job"} > 3600 * 3.5
    annotations:
      description: mybatchjob last succeeded {{humanizeDuration $value}} ago
```
