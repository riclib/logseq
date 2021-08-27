---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/sending-alert-notifications-to-multiple-destinations
author: [[Brian Brazil]] 
---
# Sending alert notifications to multiple destinations

> Usually the Prometheus Alertmanager will send a given notification to only one destination. What if you want it to go to both Slack and Pagerduty though?

---
Usually the Prometheus [Alertmanager](https://prometheus.io/docs/alerting/alertmanager/) will send a given notification to only one destination. What if you want it to go to both [Slack](https://www.robustperception.io/using-slack-with-the-alertmanager/) and [Pagerduty](https://www.robustperception.io/using-pagerduty-with-the-alertmanager/) though?

The first and least error prone way is to create a receiver that goes to both destinations:

```yaml
route:
 receiver: slack_pagerduty

receivers:
  - name: slack_pagerduty
    slack_configs:
      - api_url: THE_WEBHOOK_URL
        channel: '#general'
    pagerduty_configs:
      - service_key: AN_INTEGRATION_KEY
```
Similarly as `slack_configs` and friends take lists, you can send to multiple Slack channels:

```yaml
route:
 receiver: multi_slack

receivers:
- name: multi_slack
   slack_configs:
     - api_url: THE_WEBHOOK_URL
       channel: '#general'
     - api_url: THE_WEBHOOK_URL
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
    slack_configs:
      - api_url: THE_WEBHOOK_URL
        channel: '#general'
  - name: pagerduty
    pagerduty_configs:
      - service_key: AN_INTEGRATION_KEY
```

Usually when a route matches an alert, that's it and there's no consideration of subsequent siblings of that route. `continue` changes this behaviour, making the route match and also continuing to the next sibling with the usual route matching logic.

`continue` is primarily useful if you want to send all alerts somewhere, such as a webhook to log all alerts.
