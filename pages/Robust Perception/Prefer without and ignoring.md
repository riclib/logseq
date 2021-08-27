---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus
source: https://www.robustperception.io/prefer-without-and-ignoring
author: [[Brian Brazil]]
---
> Which of by/without and on/ignoring should you use? you should prefer using `without` and `ignoring` over `by` and `on` in order to future proof your PromQL expressions and make them easier to reuse by others.

# Prefer without and ignoring | Prometheus Monitoring Experts

Which of by/without and on/ignoring should you use?

When writing a given bit of PromQL you know what labels you want in the output of an aggregation, so why not put them in the `by` ? Similarly when doing vector matching and using `on`. So why ever use `without` and `ignoring`?

Application architectures and deployment models are rarely constant. While today you know what labels you want in the output of graph, what if tomorrow you add a new target label such as if targets are now split out by `env="dev"` and `env="prod"`? You'd now have to go to every single place that you've hardcoded the list of relevant labels in a `by`/`on` and extend them by the new label. Similarly for alerts, it's generally useful to have any labels that are available end up in the ultimate alert that is sent on to the Alertmanager, and subsequently in a notification. If you use `by` in your aggregations, you don't gain automatic access to those labels in your alerts.

This is considering just one deployment of an application that changes over time. What labels are relevant is going to vary across organisations and deployments, so you can't easily reuse rules, alerts, and dashboards. Instead you'd need to tweak them in line with what the target label taxonomy is for that particular deployment, and reapply those tweaks as the upstream rules, alerts, and dashboards change.

This is where `without` and `ignoring` come in. They allow specifying a list of what labels to exclude, so you only need to indicate the labels that you are aggregating away. What other additional target labels may be present are not relevant, so rules, alerts and dashboards written with them don't need tweaking to keep up to date with architectural changes that affect target label taxonomy. This also enables easy sharing of configuration without having to resort to templating and the like to deal with different target label setup.

However there are places where you should use `on` and `by`, and that is where there is the potential for the instrumentation labels to change. This is namely for info-style metrics, where adding new instrumentation labels over time is normal and not considered breaking. _In that case_[^1] you'd probably be using `group_left` with `on` so you wouldn't lose all the target labels on the left hand side of the expression and can still reuse expressions across different deployments. Unfortunately this doesn't work for an expression like  `count by (release, job)(node_uname_info{job="node"})` which is counting how many of each kernel version is running, as there are potentially both unknown target and instrumentation labels. Expressions like these however tend to only be a fairly small portion of a setup, so the burden of updating them is much smaller.

In summary, you should prefer using `without` and `ignoring` over `by` and `on` in order to future proof your PromQL expressions and make them easier to reuse by others.
-
- [^1] [[How to have labels for machine roles]]