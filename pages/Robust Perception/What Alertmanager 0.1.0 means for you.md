---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/what-alertmanager-0-1-0-means-for-you
author: [[Brian Brazil]] 
---
# What Alertmanager 0.1.0 means for you

> The rewrite of the Alertmanger had its first release earlier this week after a beta period. While still providing the core functionality of deduplicating alerts, silencing, inhibiting and sending notifications, there's several major changes from the previous version.

---
The rewrite of the Alertmanger had its first release [earlier this week](https://groups.google.com/forum/#!topic/prometheus-developers/fDCTkEUqmMc) after a beta period. While still providing the core functionality of deduplicating alerts, silencing, inhibiting and sending notifications, there's several major changes from the previous version.

The most obvious change in Alertmanager 0.1.0 is that the config file format is YAML rather than protocol buffers, however more important is that the structure of the file has changed. How alerts are processed is now handled by a tree of routing rules, where previously it was just a list. This means that each team could control how their alerts are routed in their sub-tree, without affecting other teams. This configuration fragment provides a simple example:

```yaml
route:
 receiver: 'fallback'
 # Settings for this route, and defaults for children.
 group_by: ['alertname', 'datacenter']

 # The children routes.
 routes:
   # This is a route for team X.
  - match:
      service: foo
    receiver: team-X-email
    # And a sub-route for team X/
    routes:
    - match:
        severity: page
      receiver: team-X-pager

  # This is a route for team Y.
  - match:
      service: bar
    receiver: team-Y-email
```

The second big change is that all alert notifications are customisable via templates. While the defaults should be fine for many use cases, if you want to change how your emails or pages look to better suit your organisation you now can.

The third change is that multiple alerts can now be aggregated into a single notification. This means for example you can get one email for all the alerts for an entire datacenter.

You should be aware that while the 0.1.0 version of the new Alertmanger works with any Prometheus version, Prometheus 0.17.0 onwards (which at the time of writing is still in beta) will only work with the new Alertmanager.

The new Alertmanager continues to be improved, notification methods are being added and features introduced. To start using it there's the [official release](https://github.com/prometheus/alertmanager/releases).

For more information, check out the [documentation](https://prometheus.io/docs/alerting/configuration/).
