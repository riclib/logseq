---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/controlling-the-instance-label
author: [[Brian Brazil]] 
---
# Controlling the instance label

> In a previous post I said that rather than adding another label such as host or alias to a target to give it a useable name, you should instead change the instance label. Let's see how you do that.

---
In a [previous post](http://www.robustperception.io/target-labels-are-for-life-not-just-for-christmas/) I said that rather than adding another label such as `host` or `alias` to a target to give it a useable name, you should instead change the `instance` label. Let's see how you do that.

In [Prometheus](https://prometheus.io/) the `instance` label uniquely identifies a target within a job. It may be a DNS name but commonly it's just a host and port such as `10.3.5.2:9100`.  That could be fine, but sometimes you'd like a more meaningful value on your graphs and dashboards. The good news is there's a way to do with without polluting your target labels with label like `host` or `alias`.

The trick to this is that the `instance` label of a target isn't what Prometheus connects to, it actually connects to what's in the `__address__` label. This means you can change the `instance` label to any value you like, and Prometheus will still successfully scrape the target.

Why does it seem as though the `instance` label is what Prometheus connects to? The answer is that the `instance` label is one of the two special target labels that must have a value (the other being `job`). Accordingly if no `instance` label is present after service discovery and relabelling, it'll be set to the value of `__address__`.

Enough theory, let's look at an example.

Let's say you are on EC2 and wanted to use the `Name` tag rather than the private IP and port you'd get by default with [EC2 service discovery](https://prometheus.io/docs/operating/configuration/#%3Cec2_sd_config%3E). You could use a relabel rule like:

```yaml
relabel_configs:
  - source_labels: [__meta_ec2_tag_Name]
    target_label: instance
```
You could go a step further and use the public rather than private IP, while also setting the `instance` label:

```yaml
relabel_configs:
 - source_labels: [__meta_ec2_public_ip]
   regex:  '(.*)'             # This is the default value.
   target_label: __address__
   replacement: '${1}:9100'   # Have to specify a port too.
 - source_labels: [__meta_ec2_tag_Name]
   target_label: instance
```
You don't always need to use relabelling to get a more meaningful name. Providing DNS names as the `__address__`/`instance` for service discovery is supported, and can work well for simpler use cases.

For more detail on how labels flow, check out [Life of a Label](http://www.robustperception.io/life-of-a-label/).
