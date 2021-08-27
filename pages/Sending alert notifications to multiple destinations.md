---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus
source: https://www.robustperception.io/sending-alert-notifications-to-multiple-destinations
author: [[Brian Brazil]]
---

> Usually the Prometheus Alertmanager will send a given notification to only one destination. What if you want it to go to both Slack and Pagerduty though?

# Sending alert notifications to multiple destinations – Robust Perception
- DONE Review #clipping Sending alert notifications to multiple destinations – Robust Perception
  done:: 1628194801215
  
  Usually the Prometheus [Alertmanager](https://prometheus.io/docs/alerting/alertmanager/) will send a given notification to only one destination. What if you want it to go to both Slack[^1] and Pagerduty[^2] though?
  
  The first and least error prone way is to create a receiver that goes to both destinations:
  
  ```yaml
  route:
   receiver: slack\_pagerduty
  
  receivers:
  - name: slack\_pagerduty
  	slack\_configs:
   - api\_url: THE\_WEBHOOK\_URL
  channel: '#general'
  	pagerduty\_configs:
   - service\_key: AN\_INTEGRATION\_KEY
  ```
  
  Similarly as `slack_configs` and friends take lists, you can send to multiple Slack channels:
  
  ```yaml
  route:
   receiver: multi\_slack
  
  receivers:
  - name: multi\_slack
   slack\_configs:
  - api\_url: THE\_WEBHOOK\_URL
    channel: '#general'
  - api\_url: THE\_WEBHOOK\_URL
    channel: '#alerts'
  ```
  
  The second approach is to use `continue`. This requires some care, as you need to remember to apply any changes to one route to the other routes:
  
  ```yaml
  route:
   receiver: slack  # Fallback.
   routes:
   - match:
    severity: page
  continue: true
  receiver: slack
   - match:
    severity: page
  receiver: pagerduty
  
  
  receivers:
  - name: slack
  	slack\_configs:
   - api\_url: THE\_WEBHOOK\_URL
  channel: '#general'
  - name: pagerduty
  	pagerduty\_configs:
   - service\_key: AN\_INTEGRATION\_KEY
  ```
  
  Usually when a route matches an alert, that's it and there's no consideration of subsequent siblings of that route. `continue` changes this behaviour, making the route match and also continuing to the next sibling with the usual route matching logic.
  
  `continue` is primarily useful if you want to send all alerts somewhere, such as a webhook to log all alerts.
  
  [^1] [[Using Slack with the Alertmanager]]