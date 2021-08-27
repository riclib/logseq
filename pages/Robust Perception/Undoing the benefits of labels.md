---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/undoing-the-benefits-of-labels
author: [[Brian Brazil]] 
---
# Undoing the benefits of labels

> It can seem like a good idea to use recording rules to make more explicit the content of a time series, particularly for those not used to labels. However this usually leads to confusing names and losing the benefits of labels.

---
It can seem like a good idea to use recording rules to make more explicit the content of a time series, particularly for those not used to labels. However this usually leads to confusing names and losing the benefits of labels.

Every so often you come across [recording rules](https://prometheus.io/docs/querying/rules/) of the form:

```
node_disk_bytes_read:sda = rate(node_disk_bytes_read{device="sda"}[5m])
node_disk_bytes_read:sdb = rate(node_disk_bytes_read{device="sdb"}[5m])
node_disk_bytes_read:sdc = rate(node_disk_bytes_read{device="sdc"}[5m])
node_md_disks:md0 = node_md_disks{device="md0"}
```

This is making work for yourself, while also losing one of the biggest wins of Prometheus - labels.

With rules like this every time there's a new label value you need to update your rules. You cannot manipulate these style of time series en-masse, as they don't share a metric name. You may also be introducing additional race conditions and graph artifacts. The name also makes it unclear exactly what the time series represents.

If you want to refer to the number of disks in `md0` the canonical way to do that is:

```
node_md_disks{device="md0"}
```

It's about the same length, clarifies what `md0` is and there's no need for recording rules!

If you are doing some computation and want to record the values, it's best to do it all at once:

```
instance_device:node_disk_bytes_read:rate5m = rate(node_disk_bytes_read{job="node"}[5m])
```

This doesn't need updating as you gain new disks, and specifying `job="node"` means you won't accidentally apply this rule to other unrelated jobs that happen to also export a metric by this name.

The `instance_device:node_disk_bytes_read:rate5m` name has further advantages. The `instance_device` tells you exactly which labels are in play, making it easier to visually inspect expressions for correct label handling. The `rate5m` at the end lets you know what calculations have been performed on it.

You may be tempted to encode other information into recording rules names such as some token to indicate whether to federate individual time series. This is to be avoided as it is unrelated to the identity of the data and thus may change over time, similar to how [target labels should be kept constant](http://www.robustperception.io/target-labels-are-for-life-not-just-for-christmas/). In general, a recording rule should either have zero colons or two colons.

The full conventions for naming recording rules are part of the Prometheus [best practices](https://prometheus.io/docs/practices/rules/). With everyone following the same naming scheme everyone wins, as at a glance we can all understand and reuse each others' rules!
