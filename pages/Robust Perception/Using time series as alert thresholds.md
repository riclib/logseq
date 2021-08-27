---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/using-time-series-as-alert-thresholds
author: [[Brian Brazil]] 
---
# Using time series as alert thresholds

> Usually alert thresholds are hardcoded in the alert. In more sophisticated setups, it would be useful for it to be parameterised based on another time series.

---
Usually alert thresholds are hardcoded in the alert. In more sophisticated setups, it would be useful for it to be parameterised based on another time series.

When your team is the only one creating alerts for your Prometheus, having the thresholds directly in the alert is the easiest way to do things. If however other groups want the same alerts in your Prometheus, but with different thresholds, this can be a bit much boilerplate. The good news is that you can use PromQL to make this easier.

To do so create a time series with the threshold using recording rules, or you could have it come from an exporter. This threshold time series can then be compared against the time series of interest, using `group_left` to handle any potential many-to-one matching. Here we presume that the `team` label is what you wish to match on, but it could be any label such as `instance`, `job` or `env`:
```yaml
groups:
- name: example
  rules:
  - record: something_too_high_threshold
    expr: 200
    labels:
       team: foo
  - record: something_too_high_threshold
    expr: 400
    labels:
      team: bar
  - alert: SomethingTooHigh
    expr: |
      # Alert based on per-team thresholds.
        something
      > on (team) group_left
        something_too_high_threshold

You could also provide a default, so only those teams wishing to override it need to configure a threshold. Here the default is 42:

- alert: SomethingTooHigh
  expr: |
    # Alert based on per-team thresholds, with a default.
      something 
    > on (team) group_left() 
      (
          something_too_high_threshold 
        or on(team) 
          count by (team)(something) * 0 + 42
      )
```

The `count by (team)(something) * 0 + 42` will produce a time series with the value 42 for every unique `team` label value in `something`.

This approach can be combined with the technique to [use labels to direct email notifications](https://www.robustperception.io/using-labels-to-direct-email-notifications/):, where the threshold time series also includes a label that can subsequently be used by the alertmanager:

```yaml
groups:
- name: example
  rules:
  - record: something_too_high_threshold
    expr: 200
    labels:
       team: foo
       email_to: foo@example.org
  - alert: SomethingTooHigh
    expr: |
      # Alert based on per-team thresholds, copying over email_to.
        something 
      > on (team) group_left(email_to) 
        something_too_high_threshold
```

With any of these alerting rules your users only need to worry about adding new threshold time series, making things easier and less error prone!

