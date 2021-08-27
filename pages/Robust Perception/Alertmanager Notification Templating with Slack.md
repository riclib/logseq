---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/alertmanager-notification-templating-with-slack
author: [[Brian Brazil]] 
---
# Alertmanager Notification Templating with Slack

> We've already looked at how to setup Slack with the Alertmanager, and saw the default notification. Wouldn't it be nice to customise it?

---
We've already looked at how to setup [Slack with the Alertmanager](http://www.robustperception.io/using-slack-with-the-alertmanager/), and saw the default notification. Wouldn't it be nice to customise it?

I'm going to presume we already have a working Slack and Alertmanager setup.

Let's say we have two alerts firing with the labels `{alertname="MyAlert",instance="host1"}` and `{alertname="MyAlert",instance="host2"}` that are going to our Slack receiver, and we want to display all the instance labels.

A key idea with the Alertmanager is that a single notification can contain more than one alert. This allows for a great reduction in notification spam, and is what the `group_by` in the `alertmanger.yml` controls.

This means we can't simply use `.Labels.instance` to get to the instance label, as each alert that will be part of the notification has its own instance labels. Instead we need a for loop using the `range` action of [Go templates](https://golang.org/pkg/text/template/#hdr-Actions).

```yaml
receivers:
  - name: slack_general
    slack_configs:
    - api_url: https://hooks.slack.com/services/XXXXXXXXXXXXXXX/XXXXXXXX
      channel: '#general'
      text: 'Instances: {{ range .Alerts }}{{ .Labels.instance }} {{ end }}'
```
This will produce a notification like:

[![screenshot-from-2016-09-18-154555](http://www.robustperception.io/wp-content/uploads/2016/09/Screenshot-from-2016-09-18-154555.png)](http://www.robustperception.io/wp-content/uploads/2016/09/Screenshot-from-2016-09-18-154555.png)

`Annotations` can be accessed similarly to `Labels`. You now know enough to do some basic notification templating!
