---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/identifying-expensive-alerting-rules
author: [[Conor Broderick]] 
---
> Since Prometheus 2.1 there is a feature to view alerting rule evaluation times in the rules UI. In this blogpost we'll see an example of how this can be used to identify an expensive rule expression.

# Identifying expensive alerting rules


Since Prometheus 2.1 there is a feature to view alerting rule evaluation times in the rules UI. In this blogpost we'll see an example of how this can be used to identify an expensive rule expression.

Having run [Prometheus 2.1](https://github.com/prometheus/prometheus/releases/tag/v2.1.0) with the following configuration and alerting rules for a few days to obtain enough data:

prometheus.yml

global:
  scrape\_interval: 5s
  evaluation\_interval: 5s

scrape\_configs:
  - job\_name: 'node'
    static\_configs:
      - targets:
        - '127.0.0.1:9100'

rule\_files:
  - alerting.rules

alerting.rules

groups:
- name: test.rules
  rules:
  - alert: LatencyTooHigh
    expr: job:request\_latency\_seconds:mean5m{job="promtheus"} > 0.5 and ON(job) job:requests:rate5m{job="prometheus"} > 100
  - alert: ExpensiveAlertingRule
    expr: rate(prometheus\_rule\_evaluation\_duration\_seconds\_count{job="prometheus", rule\_type="recording"}\[1d\]) > 1

we can head over to the new rules UI and see which rules are taking the longest to evaluate:

[![](https://www.robustperception.io/wp-content/uploads/2018/02/Screen-Shot-2018-02-14-at-17.14.11-1200x268.png)](https://www.robustperception.io/wp-content/uploads/2018/02/Screen-Shot-2018-02-14-at-17.14.11.png)

As we can see from the grid above, the problematic alerting rule which is taking a rate over a long period of time of day is rather expensive and taking a lot of time in order to be evaluated (more than x8 times our other rule).

This is useful as we can now pinpoint which of our rules are expensive without having to debug the alerting rules ourselves which may require deeper knowledge of PromQL and the data being queried.
