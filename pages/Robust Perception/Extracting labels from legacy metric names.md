---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/extracting-labels-from-legacy-metric-names
author: [[Brian Brazil]] 
---
# Extracting labels from legacy metric names

> When metrics come from another system they often don't have labels. metric_relabel_configs offers one way around that.

---
When metrics come from another system they often don't have labels. `metric_relabel_configs` offers one way around that.

Let's say that you were using a legacy system that produces metrics for JVM memory that looked like:
```
memory_pools_PS_Eden_Space_committed
memory_pools_PS_Eden_Space_max
memory_pools_PS_Eden_Space_used
memory_pools_PS_Old_Gen_committed
memory_pools_PS_Old_Gen_max
memory_pools_PS_Old_Gen_used
memory_pools_PS_Survivor_Space_committed
memory_pools_PS_Survivor_Space_max
memory_pools_PS_Survivor_Space_used
```
It looks like `PS_Eden_Space`, `PS_Old_Gen` and `PS_Survivor_Space` would make sense as labels, and a unit would also be helpful.

While it'd be best to have the metrics correct in the first place (in this case by using the Java client's [DefaultExports](https://github.com/prometheus/client_java#included-collectors)), that's not always possible. What we can do instead is use `metric_relabel_configs` to extract the label and adjust the metric name. As you'll recall, `metric_relabel_configs` [are applied after the scrape](https://www.robustperception.io/relabel_configs-vs-metric_relabel_configs/).
```yaml
scrape_configs:
  job_name: my_job
  # Usual fields go here to specify targets.
  metric_relabel_configs:
  - source_labels: [__name__]
    regex: '(memory_pools)_(.*)_(w+)'
    replacement: '${2}'
    target_label: pool
  - source_labels: [__name__]
    regex: '(memory_pools)_(.*)_(w+)'
    replacement: '${1}_${3}_bytes'
    target_label: __name__
```
`__name__` is the special label which contains the metric name. As we happen to know the units are bytes, we can also add that in. This will create the metrics:
```
memory_pools_committed_bytes{pool="PS_Eden_Space"}
memory_pools_max_bytes{pool="PS_Eden_Space"}
memory_pools_used_bytes{pool="PS_Eden_Space"}
memory_pools_committed_bytes{pool="PS_Old_Gen"}
memory_pools_max_bytes{pool="PS_Old_Gen"}
memory_pools_used_bytes{pool="PS_Old_Gen"}
memory_pools_committed_bytes{pool="PS_Survivor_Space"}
memory_pools_max_bytes{pool="PS_Survivor_Space"}
memory_pools_used_bytes{pool="PS_Survivor_Space"}
```
Much better!
