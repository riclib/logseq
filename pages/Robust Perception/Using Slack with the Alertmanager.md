---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/using-slack-with-the-alertmanager
author: [[Brian Brazil]] 
---
# Using Slack with the Alertmanager

> Previously we looked at how to use the Prometheus Alertmanager with Pagerduty. The Alertmanager supports more than just sending pages, there's integrations with popular chat applications too. Let's look at how to integrate with Slack.

---
Previously we looked at how to use the Prometheus [Alertmanager with Pagerduty](http://www.robustperception.io/using-pagerduty-with-the-alertmanager/). The Alertmanager supports more than just sending pages, there's integrations with popular chat applications too. Let's look at how to integrate with [Slack](https://slack.com/).

## Slack Setup

The Alertmanager uses the [Incoming Webhooks](https://api.slack.com/incoming-webhooks) feature of Slack, so first we need to set that up.

Go to the [Incoming Webhooks page in the App Directory](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks) and click "Install" (or "Configure" and then "Add Configuration" if it's already installed):

[![adding-webhook-app](http://www.robustperception.io/wp-content/uploads/2016/03/adding-webhook-app.png)](http://www.robustperception.io/wp-content/uploads/2016/03/adding-webhook-app.png)

You can then configure your new webhook. Choose the default channel to post to, and then add the integration:

[![configure-webhook](http://www.robustperception.io/wp-content/uploads/2016/03/configure-webhook.png)](http://www.robustperception.io/wp-content/uploads/2016/03/configure-webhook.png)

This will then give us the Webhook URL we need:

[![edit-config](http://www.robustperception.io/wp-content/uploads/2016/03/edit-config.png)](http://www.robustperception.io/wp-content/uploads/2016/03/edit-config.png)

Let's download, configure and run an Alertmanger:

```shell
WEBHOOK_URL=XXXX  # Substitute in your Webhook URL here.
wget https://github.com/prometheus/alertmanager/releases/download/v0.8.0/alertmanager-0.8.0.linux-amd64.tar.gz
tar -xzf alertmanager-0.8.0.linux-amd64.tar.gz
cd alertmanager-*

cat <<EOF > alertmanager.yml
route:
 group_by: [cluster]
 # If an alert isn't caught by a route, send it slack.
 receiver: slack_general
 routes:
  # Send severity=slack alerts to slack.
  - match:
      severity: slack
    receiver: slack_general

receivers:
- name: slack_general
  slack_configs:
  - api_url: $WEBHOOK_URL
    channel: '#general'
EOF
./alertmanager &
```

That's all now setup, and you can see your firing alerts in Slack:

[![notification-in-slack](http://www.robustperception.io/wp-content/uploads/2016/03/notification-in-slack.png)](http://www.robustperception.io/wp-content/uploads/2016/03/notification-in-slack.png)

The flexibility the Prometheus Alertmanager offers means that each team can route alerts to their own Slack channels, and customise the messages.
