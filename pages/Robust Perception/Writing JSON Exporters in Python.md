---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus python
source: https://www.robustperception.io/writing-json-exporters-in-python
author: [[Brian Brazil]] 
---
# Writing JSON Exporters in Python

> A common question is is there a way to ingest JSON metrics from a random system into Prometheus? It's not possible to extract useful metrics from an arbitrary JSON blob, so that's not something the can be offered out of the box. However it's easy to write an exporter in Python to produce meaningful metrics.

---
A common question is is there a way to ingest JSON metrics from a random system into [Prometheus](https://prometheus.io/)? It's not possible to extract useful metrics from an arbitrary JSON blob, so that's not something the can be offered out of the box. However it's easy to write an exporter in Python to produce meaningful metrics.

**_Update_**: The Python client has a new API that's easier to use for custom collectors, see [Writing a Jenkins exporter in Python](http://www.robustperception.io/writing-a-jenkins-exporter-in-python/) for and example of how to use it.

Let's say you have a service called "SVC" that produces JSON output over HTTP.
```json
{
  "requests_handled": 14324,
  "requests_duration_milliseconds": 1235257,
  "request_failures": 7,
  "documents_loaded": {
    "fast": 7,
    "slow": 60
  }
}
```

To start, let's install the Prometheus Python client, and the Requests library..

pip install prometheus_client requests

Then we can write a simple exporter. Put the following in a file called `json_exporter.py`:
```python
from prometheus_client import start_http_server, Metric, REGISTRY
import json
import requests
import sys
import time

class JsonCollector(object):
  def __init__(self, endpoint):
    self._endpoint = endpoint
  def collect(self):
    # Fetch the JSON
    response = json.loads(requests.get(self._endpoint).content.decode('UTF-8'))

    # Convert requests and duration to a summary in seconds
    metric = Metric('svc_requests_duration_seconds',
        'Requests time taken in seconds', 'summary')
    metric.add_sample('svc_requests_duration_seconds_count',
        value=response['requests_handled'], labels={})
    metric.add_sample('svc_requests_duration_seconds_sum',
        value=response['requests_duration_milliseconds'] / 1000.0, labels={})
    yield metric

    # Counter for the failures
    metric = Metric('svc_requests_failed_total',
       'Requests failed', 'summary')
    metric.add_sample('svc_requests_failed_total',
       value=response['request_failures'], labels={})
    yield metric

    # Metrics with labels for the documents loaded
    metric = Metric('svc_documents_loaded', 'Requests failed', 'gauge')
    for k, v in response['documents_loaded'].items():
      metric.add_sample('svc_documentes_loaded', value=v, labels={'repository': k})
    yield metric


if __name__ == '__main__':
  # Usage: json_exporter.py port endpoint
  start_http_server(int(sys.argv[1]))
  REGISTRY.register(JsonCollector(sys.argv[2]))

  while True: time.sleep(1)
```

This can now be run as
```shell
python json_exporter.py 1234 http://host/path/to/metrics.json
```
