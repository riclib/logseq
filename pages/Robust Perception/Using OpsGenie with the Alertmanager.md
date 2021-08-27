---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/using-opsgenie-with-the-alertmanager
author: [[Brian Brazil]] 
---
# Using OpsGenie with the Alertmanager

> The Alertmanager has integrations to a variety of popular notification mechanisms. Let's see how easy it is to hook it in to OpsGenie.

---
The [Alertmanager](https://prometheus.io/docs/alerting/alertmanager/) has integrations to a variety of popular notification mechanisms. Let's see how easy it is to hook it in to [OpsGenie](https://www.opsgenie.com/).

## OpeGenie Setup

First we need to create an integration in OpsGenie, and obtain an API key.

From the side menu, go to the "Integrations" page in OpsGenie:

[![screenshot_2016-11-18_16-57-37](https://www.robustperception.io/wp-content/uploads/2016/11/Screenshot_2016-11-18_16-57-37.png)](https://www.robustperception.io/wp-content/uploads/2016/11/Screenshot_2016-11-18_16-57-37.png)

Add a new "API" integration:

[![screenshot_2016-11-18_16-58-00](https://www.robustperception.io/wp-content/uploads/2016/11/Screenshot_2016-11-18_16-58-00.png)](https://www.robustperception.io/wp-content/uploads/2016/11/Screenshot_2016-11-18_16-58-00.png)

Note down the API Key, and save the integration:

[![screenshot_2016-11-18_16-58-23](https://www.robustperception.io/wp-content/uploads/2016/11/Screenshot_2016-11-18_16-58-23.png)](https://www.robustperception.io/wp-content/uploads/2016/11/Screenshot_2016-11-18_16-58-23.png)

## Alertmanager Setup

Let's download, configure and run an Alertmanger:

```shell
API_KEY=XXXX  # Substitute in your API key here.
wget https://github.com/prometheus/alertmanager/releases/download/v0.5.0/alertmanager-0.5.0.linux-amd64.tar.gz
tar -xzf alertmanager-*.linux-amd64.tar.gz
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
  opsgenie_configs:
  - api_key: $API_KEY
    teams: example_team   # Put in your team here
EOF
./alertmanager &
```

That’s all now setup, and you can see your firing alerts in OpsGenie:

[![screenshot_2016-11-18_17-26-22](https://www.robustperception.io/wp-content/uploads/2016/11/Screenshot_2016-11-18_17-26-22.png)](https://www.robustperception.io/wp-content/uploads/2016/11/Screenshot_2016-11-18_17-26-22.png)

The flexibility of the Prometheus Alertmanager offers means that each team can route alerts to their own team, and customise the messages.
