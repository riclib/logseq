---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/whats-the-difference-between-group_interval-group_wait-and-repeat_interval
author: [[Brian Brazil]] 
---
> In this blogpost we try and clear up some confusion by outlining the key differences between commonly confused alerting configuration options: group_interval, group_wait, and repeat_interval.

# What’s the difference between group_interval, group_wait, and repeat_interval?


In this blogpost we try and clear up some confusion by outlining the key differences between commonly confused alerting configuration options: `group_interval`, `group_wait`, and `repeat_interval`.

Before digging into these 3 Alertmanager configuration options, let's recap on some Prometheus alerting basics.  
Prometheus itself has two global clocks: `scrape_interval` and `evaluation_interval`.

The `scrape_interval` is the time between each Prometheus scrape (i.e when Prometheus is pulling data from exporters etc.), and the `evaluation_interval` is the time between each evaluation of Prometheus' alerting rules.

When a rule is evaluated, its state can be altered to be either inactive, pending, or firing.  
Following evaluation, this state is sent to the connected Alertmanager to potentially start/stop the sending of alert notifications.

This is where `group_by` comes into play.

In order to avoid continuously sending notifications for similar alerts (like the same process failing on multiple instances, nodes, and data centres), the Alertmanager may be configured to group these related alerts into one alert:

group\_by: \['alertname', 'job'\]

Instead we wait for the `group_interval` since the last notification was sent to the group, and then send all alerts firing (and any resolved alerts) to the receiver.

`group_wait` sets how long to initially wait to send a notification for a particular group of alerts.

This allows the Alertmanager to wait for an inhibiting alert to arrive or to collect more initial alerts for the same group. It essentially buffers alerts from Prometheus sent to the Alertmanager that are grouped by the same labels:

group\_by: \['alertname', 'job'\]
group\_wait: 45s # Usually set between ~0s to a few minutes.

While this reduces noisy alerts and saves the people receiving them some headache, it may introduce longer delays in receiving said alert notifications.

Another issue we must consider is that we'll receive the same grouped alert notification again next time the rules are evaluated.

This is where we use `group_interval`.

`group_interval` dictates how long to wait before sending notifications about new alerts that are added to a group of alerts that have been alerted on before:

group\_by: \['instance', 'job'\]
group\_wait: 45s
group\_interval: 10m # Usually ~5 mins or more.

So where does `repeat_interval` fit into all of this?

Simply put, `repeat_interval` is used to determine the wait time before a firing alert that has already been successfully sent to the receiver is sent again.

To summarise:

`group_wait`

How long to wait to buffer alerts of the same group before sending a notification initially.

`group_interval`

How long to wait before sending an alert that has been added to a group for which there has already been a notification.

`repeat_interval`

How long to wait before re-sending a given alert that has already been sent in a notification.

_Want expert help on Prometheus configuration? [Contact us](mailto:prometheus@robustperception.io)._

Published by Conor Broderick in [Posts](https://www.robustperception.io/category/posts)
