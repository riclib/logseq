---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus yaml
source: https://www.robustperception.io/using-labels-to-direct-email-notifications
author: [[Brian Brazil]] 
---
# Using labels to direct email notifications

> A handy feature of the Alertmanager is that almost all notification fields are templatable. This can be used to route emails based on labels.

---
A handy feature of the [Alertmanager](https://github.com/prometheus/alertmanager) is that almost all notification fields are templatable. This can be used to route emails based on labels.

If you are sending email notifications directly to customers, it'd be useful not to have to create a route and receiver per customer in the Alertmanager. This can be useful whether you are a hosting company providing alerting on the machines you rent out, or the provider of a shared service within a company.

I'm going to presume you already have a working email notification setup, such as was shown previously with [Gmail](https://www.robustperception.io/sending-email-with-the-alertmanager-via-gmail/).

We start at Prometheus. We'll need to get the target email address into an alert label. Let's say you have constructed time series looking like:

```
mymetric{job="myjob",email_to="foo@example.org",instance="host1"}
mymetric{job="myjob",email_to="bar@example.org",instance="host2"}
```

Then you can write an alert which propagates this `email_to` label such as:

```yaml
groups:
- name: test.rules
  rules:
  - alert: MyAlert
    expr: mymetric > 1
```

Now in the Alertmanager we need to do two things. We need to make sure each unique email address gets its own notification via grouping, and we need to make it then go to that email address.

```yaml
route:
  group_by: [email_to]
  receiver: email_router

receivers:
- name: email_router
  email_configs:
  - to: "{{ .GroupLabels.email_to }}"
    from: alerts@example.org
    # Add whatever other settings you need to talk to your SMTP server
    headers:
      subject: "You have {{ .Alerts.Firing | len }} firing alerts"
    html: |
      Hi {{ .GroupLabels.email_to }},

      <p>
      You have the following firing alerts:
      <ul>
      {{ range .Alerts }}
      <li>{{.Labels.alertname}} on {{ .Labels.instance }}</li>
      {{ end }}
      </ul>
      </p>
```

Due to `email_to` being in the `group_by`, we can access it via `.GroupLabels`.  By contrast as alerts with many different `Alertname` and `instance` labels could be in this one notification, we have to iterate over them to list t hem all.

This technique works with other receivers too, so you could use this to send to customer's Pagerduty services or Slack channels for example.
