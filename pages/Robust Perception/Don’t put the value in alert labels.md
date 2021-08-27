---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/dont-put-the-value-in-alert-labels
author: [[Brian Brazil]] 
---
> The labels of an alert are its identity, so you have to be a little careful what you put in there.

# Don’t put the value in alert labels


The labels of an alert are its identity, so you have to be a little careful what you put in there.

Alerting rules have both `labels` and `annotations` fields, and when configuring them on the Prometheus side there's no difference between them. However their semantics and how both Prometheus and the Alertmanager deal with them are quite different. Consider for example this alert:

groups:
- name: example
  rules:
  - alert: ExampleAlert
    expr: metric > 10
    for: 5m
    labels:
      severity: page
      value: "{{ $value }}"
    annotations:
      summary: "Instance {{ $labels.instance }}'s metric is too high"

This may seem okay at a first glance, however it's likely this alert will never fire.

The reason is that the `labels` include a label called `value`, which will usually vary from one evaluation from the next. The effect of this is that each evaluation will see a brand new alert, and treat the previous one as no longer firing. This is as the labels of an alert define its identity, and thus the `for` will never be satisfied.

If alerts from this rule did make it to the Alertmanager, it'd likely only exist briefly and either not result in a notification due to `group_wait`/`group_interval` or spam you every `group_interval` as the Alertmanager's throttling, routing, de-duplication, and grouping is all based on alert identity.

If you want to have the value of the alert expression or anything else that can vary from one evaluation of an alert instance to another, you should use `annotations` instead. For example:

groups:
- name: example
  rules:
  - alert: ExampleAlert
    expr: metric > 10
    for: 5m
    labels:
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }}'s metric is {{ $value }}"

There are valid uses cases for using templating in alerting rule `labels`, however they're usually advanced versions of [using labels to direct email notifications](https://www.robustperception.io/using-labels-to-direct-email-notifications) and quite rare.

_Unsure how to get the alert notifications you want? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
