---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/using-hipchat-with-the-alertmanager
author: [[Brian Brazil]] 
---
# Using HipChat with the Alertmanager

> Today let's look at how to integrate HipChat with the Prometheus Alertmanager.

---
Today let's look at how to integrate [HipChat](https://www.hipchat.com/) with the Prometheus [Alertmanager](https://prometheus.io/docs/alerting/alertmanager/).

## HipChat setup

First we need to configure a Build Your Own integration for the room.

From the web app, go to the room and click "Configure Integrations":

[![](http://www.robustperception.io/wp-content/uploads/2016/06/Screenshot-from-2016-06-09-131235.png)](http://www.robustperception.io/wp-content/uploads/2016/06/Screenshot-from-2016-06-09-131235.png)

Next click "Build your own integration":

[![](http://www.robustperception.io/wp-content/uploads/2016/06/Screenshot-from-2016-06-09-131416.png)](http://www.robustperception.io/wp-content/uploads/2016/06/Screenshot-from-2016-06-09-131416.png)

Give it a name, and click "Create":

[![](http://www.robustperception.io/wp-content/uploads/2016/06/Screenshot-from-2016-06-09-131555.png)](http://www.robustperception.io/wp-content/uploads/2016/06/Screenshot-from-2016-06-09-131555.png)

With the integration created, note the room id, and the auth token:

[![](http://www.robustperception.io/wp-content/uploads/2016/06/Screenshot-from-2016-06-09-131745.png)](http://www.robustperception.io/wp-content/uploads/2016/06/Screenshot-from-2016-06-09-131745.png)

## Alertmanager setup

Let's download, configure and run an Alertmanger:

```
ROOM_ID=XXXX            # Substitute in your room id here.
AUTH_TOKEN=XXXXXXXXXXX  # Substitute in your auth token here.
wget https://github.com/prometheus/alertmanager/releases/download/v0.8.0/alertmanager-0.8.0.linux-amd64.tar.gz
tar -xzf alertmanager-0.8.0.linux-amd64.tar.gz
cd alertmanager-*

cat <<EOF > alertmanager.yml
route:
 group_by: [cluster]
 # If an alert isn't caught by a route, send it to hipchat.
 receiver: team-hipchat
 routes:
  # Send severity=hipchat alerts to hipchat.
  - match:
      severity: hipchat
    receiver: team-hipchat

receivers:
- name: team-hipchat
  hipchat_configs:
  - room_id: $ROOM_ID
    auth_token: $AUTH_TOKEN
EOF
./alertmanager &
```

That's all now setup, and you'll see firing alerts in HipChat:

[![](http://www.robustperception.io/wp-content/uploads/2016/06/Screenshot-from-2016-06-09-133042.png)](http://www.robustperception.io/wp-content/uploads/2016/06/Screenshot-from-2016-06-09-133042.png)
