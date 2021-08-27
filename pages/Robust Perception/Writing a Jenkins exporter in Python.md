---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/writing-a-jenkins-exporter-in-python
author: [[Brian Brazil]] 
---
# Writing a Jenkins exporter in Python

> I previously talked about writing JSON exporters in Python for Prometheus. Since then, the API for custom collectors in the Python client has been improved. So let's see how easy it is to create a new exporter!

---
I [previously](http://www.robustperception.io/writing-json-exporters-in-python/) talked about writing JSON exporters in Python for Prometheus. Since then, the API for custom collectors in the Python client has been improved. So let's see how easy it is to create a new exporter!

The [new API](https://github.com/prometheus/client_python#custom-collectors) removes repetitive code and handles the structure of metrics for you. A minimal [Jenkins](https://jenkins-ci.org/) exporter would expose the last time each job succeeded, giving you just enough information to alert if it was too long ago:

```javascript
import json
import time
import urllib2
from prometheus_client import start_http_server
from prometheus_client.core import GaugeMetricFamily, REGISTRY

class JenkinsCollector(object):
  def collect(self):
    metric = GaugeMetricFamily(
        'jenkins_job_last_successful_build_timestamp_seconds',
        'Jenkins build timestamp in unixtime for lastSuccessfulBuild',
        labels=["jobname"])

    result = json.load(urllib2.urlopen(
        "http://jenkins:8080/api/json?tree="
        + "jobs[name,lastSuccessfulBuild[timestamp]]"))

    for job in result['jobs']:
      name = job['name']
      # If there's a null result, we want to export a zero.
      status = job['lastSuccessfulBuild'] or {}
      metric.add_metric([name], status.get('timestamp', 0) / 1000.0)

    yield metric

if __name__ == "__main__":
  REGISTRY.register(JenkinsCollector())
  start_http_server(9118)
  while True: time.sleep(1)
```

The core here is the `GaugeMetricFamily` which creates a Gauge and specifies labels, `add_metric` which adds a sample and then the `yield` to return the metric. I avoid calling my label `job`, as that label already has a meaning for Prometheus.

This can be expanded with more metrics. A full version of this is on [Github](https://github.com/RobustPerception/python_examples/blob/master/jenkins_exporter/jenkins_exporter.py). To run it:

pip install prometheus_client
wget https://raw.githubusercontent.com/RobustPerception/python_examples/master/jenkins_exporter/jenkins_exporter.py
python jenkins_exporter.py http://jenkins:8080/

Point the exporter at your actual jenkins server. You should now be able to see your metrics at [http://localhost:9118/metrics](http://localhost:9118/metrics).

This exporter took me about an hour to write. Most of that time was spent figuring out which data was available, and how to get it efficiently out of Jenkins. Writing the code was the easy part!

_Update: Vanesa from Lovoo has taken this example exporter and expanded it  be more fully-fledged and to cover more metrics, you can find it at [https://github.com/lovoo/jenkins_exporter](https://github.com/lovoo/jenkins_exporter)_ 

