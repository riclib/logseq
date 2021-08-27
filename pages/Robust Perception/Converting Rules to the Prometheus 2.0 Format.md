---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/converting-rules-to-the-prometheus-2-0-format
author: [[Brian Brazil]] 
---
# Converting Rules to the Prometheus 2.0 Format

> With the upcoming release of Prometheus 2.0 comes a new format for writing recording and alerting rules.

---
With the upcoming release of Prometheus 2.0 comes a new format for writing recording and alerting rules.

In this brief post we'll look at how we can use Promtool to automatically update your rules to the new format, and avoid rewriting them from scratch!

First grab and untar the latest 2.0 release binaries of Prometheus:

wget https://github.com/prometheus/prometheus/releases/download/v2.0.0-rc.1/prometheus-2.0.0-rc.1.linux-amd64.tar.gz
tar -xzf prometheus-2*

Now let's update our current rules file. Let's say we have the following in `alert.rules`:
```
ALERT HighErrorRate
  IF job:request_latency_seconds:mean5m{job="myjob"} > 0.5
  FOR 10m
  ANNOTATIONS {
    summary = "High request latency",
  }

ALERT DailyTest
  IF vector(1)
  FOR 1m
  ANNOTATIONS {
    summary = "Daily alert test",
  }
```
Now we'll use Promtool to update our current rules in `alert.rules` to the new format with the following command:
```shell
./promtool update rules alert.rules
```
This will generate a new file titled `alert.rules.yml` containing all of your rules in the new 2.x format!

In my case, my two rules ended up as the following:
```yaml

groups:
- name: alert.rules
  rules:
  - alert: HighErrorRate
    expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5
    for: 10m
    annotations:
      summary: High request latency
  - alert: DailyTest
    expr: vector(1)
    for: 1m
    annotations:
      summary: Daily alert test
```
You can now update your Prometheus 2.0 configuration to use this new rules file, and verify that they work as expected. Beware that the old format only works with Prometheus 1.x, and the new format only with 2.x!
