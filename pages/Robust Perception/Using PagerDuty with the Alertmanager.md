---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/using-pagerduty-with-the-alertmanager
author: [[Brian Brazil]] 
---
# Using PagerDuty with the Alertmanager

> The new Alertmanager has integrations to a variety of popular notification mechanisms, one of those is PagerDuty. Let's see how easy it is to hook it in.

---
The [new Alertmanager](http://www.robustperception.io/what-alertmanager-0-1-0-means-for-you/) has integrations to a variety of popular notification mechanisms, one of those is PagerDuty. Let's see how easy it is to hook it in.

## PagerDuty Setup

First we need to create a service in PagerDuty, and obtain an integration key.

Go to the "Services" page in PagerDuty:

[![](http://www.robustperception.io/wp-content/uploads/2016/03/Screenshot-010316-130000.png)](http://www.robustperception.io/wp-content/uploads/2016/03/Screenshot-010316-130000.png)

Click "+ Add New Service":

[![](http://www.robustperception.io/wp-content/uploads/2016/03/Screenshot-010316-130902.png)](http://www.robustperception.io/wp-content/uploads/2016/03/Screenshot-010316-130902.png)

Add a new service that integrates with the API directly, specifying the Escalation Policy to use:

[![](http://www.robustperception.io/wp-content/uploads/2016/03/Screenshot-010316-131210.png)](http://www.robustperception.io/wp-content/uploads/2016/03/Screenshot-010316-131210.png)

Note down the Integration Key:

[![](http://www.robustperception.io/wp-content/uploads/2016/03/Screenshot-010316-131853.png)](http://www.robustperception.io/wp-content/uploads/2016/03/Screenshot-010316-131853.png)

## Alertmanager Setup

Let's download, configure and run an Alertmanger:

```shell
INTEGRATION_KEY=XXXX  # Substitute in your integration key here.
wget https://github.com/prometheus/alertmanager/releases/download/v0.8.0/alertmanager-0.8.0.linux-amd64.tar.gz
tar -xzf alertmanager-0.8.0.linux-amd64.tar.gz
cd alertmanager-*

cat <<EOF > alertmanager.yml
route:
 group_by: [cluster]
 # If an alert isn't caught by a route, send it to the pager.
 receiver: team-pager
 routes:
  # Send severity=page alerts to the pager.
  - match:
      severity: page
    receiver: team-pager

receivers:
- name: team-pager
  pagerduty_configs:
  - service_key: $INTEGRATION_KEY
EOF
./alertmanager &
```

That's all now setup.

To test it out we'll cheat a bit and rather than using Prometheus to send the alert like you're meant to, instead do it by hand:

```shell
curl -d '[{"labels": {"alertname": "PagerDutyDemo"}}]' http://localhost:9093/api/v1/alerts
```

You'll see it as a triggered incident, which will be resolved by the Alertmanager as it's not continuing to fire:[![](http://www.robustperception.io/wp-content/uploads/2016/03/Screenshot-010316-141446.png)](http://www.robustperception.io/wp-content/uploads/2016/03/Screenshot-010316-141446.png)

The flexibility the Prometheus Alertmanager offers means that each team can have it's own PagerDuty service for its own alerts, all routed based on alert labels!
