---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/high-availability-prometheus-alerting-and-notification
author: [[Brian Brazil]] 
---
> Prometheus is architected for reliability of alerting, how do you set it up?

# High Availability Prometheus Alerting and Notification


Prometheus is [architected](https://www.robustperception.io/prometheus-and-alertmanager-architecture/) for [reliability](https://www.robustperception.io/monitoring-without-consensus/) of alerting, how do you set it up?

For a setup that can gracefully handle any machine failing, we'll need to run two Prometheus servers and two Alertmanagers. First we'll run the Alertmanagers on different machines, and setup a mesh between them:

\# On a machine named "am-1":
wget https://github.com/prometheus/alertmanager/releases/download/v0.15.3/alertmanager-0.15.3.linux-amd64.tar.gz
tar -xzf alertmanager-\*.linux-amd64.tar.gz
cd alertmanager-\*
./alertmanager --cluster.peer=am-2:9094


# On a machine named "am-2":
wget https://github.com/prometheus/alertmanager/releases/download/v0.15.3/alertmanager-0.15.3.linux-amd64.tar.gz
tar -xzf alertmanager-\*.linux-amd64.tar.gz
cd alertmanager-\*
./alertmanager --cluster.peer=am-1:9094

To verify that the Alertmanager mesh is working correctly, create a silence in one Alertmanager. If it shows up in the other, then all is well.

Next we configure the Prometheus servers to talk to the Alertmanager:

\# On a machine named "prom-1":
wget https://github.com/prometheus/prometheus/releases/download/v2.5.0/prometheus-2.0.0.linux-amd64.tar.gz
tar -xzf prometheus-\*.linux-amd64.tar.gz
cd prometheus-\*
cat > prometheus.yml << EOF
global:
  external\_labels:
    dc: europe1    
alerting:
  alert\_relabel\_configs:
    - source\_labels: \[dc\]
      regex: (.+)\\d+
      target\_label: dc
  alertmanagers:
    - static\_configs:
      - targets: \['am-1:9093', 'am-2:9093'\]
# The rest of your Prometheus config goes here as usual.
EOF
./prometheus


# On a machine named "prom-2":
wget https://github.com/prometheus/prometheus/releases/download/v2.5.0/prometheus-2.0.0.linux-amd64.tar.gz
tar -xzf prometheus-\*.linux-amd64.tar.gz
cd prometheus-\*
cat > prometheus.yml << EOF
global:
 external\_labels:
   dc: europe2   # Note that this is different only by the trailing number.
alerting:
 alert\_relabel\_configs:
 - source\_labels: \[dc\]
   regex: (.+)\\d+
   target\_label: dc
 alertmanagers:
 - static\_configs:
   - targets: \['am-1:9093', 'am-2:9093'\]
# The rest of your Prometheus config goes here as usual.
EOF
./prometheus

The key point here is that both Prometheus servers talk to both Alertmanagers.

In addition the two Prometheus servers have slightly different external labels, so their data does not conflict if remote storage is in use. We then use alert relabelling to ensure they still send identically labelled alerts, which the Alertmanager will automatically de-duplicate.

As long as one Prometheus and one Alertmanager are working and can talk to each other, alerts and notifications will get through!

_Looking for expert advice on architecting Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
