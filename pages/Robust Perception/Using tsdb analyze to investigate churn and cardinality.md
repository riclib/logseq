---
created: [[Aug 4th, 2021]]
type: #clipping
status: #reviewed
tags: prometheus 
source: https://www.robustperception.io/using-tsdb-analyze-to-investigate-churn-and-cardinality
author: [[Brian Brazil]] 
---
> The Prometheus TSDB's code base includes a tool to help you find "interesting" metrics in terms of storage performance.

# Using tsdb analyze to investigate churn and cardinality


The Prometheus TSDB's code base includes a tool to help you find "interesting" metrics in terms of storage performance.

==When it comes to Prometheus resource usage and efficiency, the important questions are around cardinality and churn==. That is how many time series you have, and how often the set of time series changes. I recently added a subcommand to the `tsdb` utility to help determine this for existing blocks. To use it, download, build, and point it at a Prometheus data directory:

``` shell
wget https://github.com/prometheus/prometheus/releases/download/v2.13.0/prometheus-2.13.0.linux-amd64.tar.gz
tar -xzf prometheus-\*
cd prometheus-\*
./tsdb analyze path/to/some/prometheus/datadir | less
```
The command can take a minute to run, so I've piped to `less` so you can browse the output when it's ready. The output has a few sections, as you can see from my home Prometheus:

```
Block path: /some/datadir/01D2D1ZHR6N6XDFRBDXJ5SSHKB
Duration: 2h0m0s
Series: 15680
Label names: 52
Postings (unique label pairs): 2110
Postings entries (total label pairs): 71352
```
The first part is an overview, including the block that was looked at, how long the block covers (usually 2 hours, as the default is to use the latest block) and then various index statistics. The series count means what you think, as does label names. The posting statistics indicates how many index entries there are.

Next up is churn in label pairs:

```
Label pairs most involved in churning:
17 job=node
11 \_\_name\_\_=node\_systemd\_unit\_state
6 instance=foo:9100
4 instance=bar:9100
3 instance=baz:9100
```
The values here are quite low and nothing to worry about (a small home system doesn't have much churn). The number is the amount of sparseness in the timeseries. For example 17 for `job=node` could mean that there were 34 time series that were missing for one hour in this two hour block, or 68 time series for half an hour etc. So the number itself is mostly useful relative to the total number of series. The items at the top of the list indicate what is churning the most, and helps pinpoint which subset of targets and metrics are worth investigating further.

Then there's churn in label names:

```
Label names most involved in churning:
21 instance
21 \_\_name\_\_
21 job
13 name
11 state
```
This works in the same way as the label pairs, except only what labels time series have (but not their values) is considered. `__name__` is the metric name which all series have, so will always be at the top and isn't much use. Similarly with `instance` and `job`, which should be present on the vast majority of series. Any remaining entries can indicate a label that has high churn.

Next up is cardinality for label pairs:

```
Most common label pairs:
12503 job=node
8060 \_\_name\_\_=node\_systemd\_unit\_state
4613 instance=foo:9100
3035 instance=bar:9100
2455 instance=baz:9100
```
Here you can see I have a small setup which is largely node exporter metrics from a handful of machines. If any individual instance, job, or other subset was responsible for a disproportionate amount of cardinality it would show here.

Then there's cardinality for labels:

```
Highest cardinality labels:
1037 name
589 \_\_name\_\_
53 ifDescr
53 le
50 ifName
```

A high value here can indicate something that may not be suitable for use as a label, and based on the other results above the `name` label from `node_systemd_unit_state` has a bit more than is wise. `__name__` here is reasonable, however if it was getting into the tens of thousands or higher that could indicate that should be a label has been put inside a metric name.

Finally, metric names:

```
Highest cardinality metric names:
8060 node\_systemd\_unit\_state
1612 node\_systemd\_unit\_start\_time\_seconds
176 node\_cpu\_seconds\_total
135 http\_request\_size\_bytes
135 http\_response\_size\_bytes
```
This is similar to the data that you can _get via PromQL_[^1], though it includes the whole block rather than now and is more efficient. This is as `analyze` is using only the index of the block, and doesn't try to read the block to get values or check for staleness.

==An advantage over any PromQL-based approach is that this can be run on blocks that Prometheus is using, as blocks are read-only==. The example output here is from a small Prometheus that's nice and healthy, you can use this tool on larger systems too to help pinpoint aspects of your setup that are costing more resources than they're worth.

(Since this post was made, the tsdb repo was merged into the prometheus repo. In 2.13.0 the tsdb tool was included in Prometheus releases, in future releases it'll be part of promtool. Instructions have been updated accordingly.)
- [^1] [[Which are my biggest metrics]]