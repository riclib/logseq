---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/federation-what-is-it-good-for
author: [[Brian Brazil]] 
---
# Federation, what is it good for?

> There's various ways Prometheus federation can be used. To ensure your monitoring is scalable and reliable, let's look at how to best use it.

---
There's various ways Prometheus [federation](https://prometheus.io/docs/operating/federation/) can be used. To ensure your monitoring is scalable and reliable, let's look at how to best use it.

There are two general cases for which federation is well suited. We'll look at each in turn.

As [previously](https://www.robustperception.io/monitoring-without-consensus/) [discussed](https://www.robustperception.io/scaling-and-federating-prometheus/), Prometheus is intended to have at least one instance per datacenter usually also with a global Prometheus for global graphing or alerting. Federation allows for pulling aggregates up the hierarchy. Usually there's only two levels to the hierarchy, but three or four aren't completely unheard of.

Let's for example say you wanted to aggregate up the total amount of memory on all your machines at a global level.

First there's part of the config file for each datacenter Prometheus:

```yaml
global: 
  external_labels:
    datacenter: eu-west-1

rule_files:
  - node.rules

scrape_configs:
  etc.
```
Note the external labels, every Prometheus in an organization should have unique external labels.

Then in the `node.rules` rules file we aggregate up to the dataenter level:

job:node_memory_MemTotal:sum = sum without(instance)(node_memory_MemTotal{job="node"})

As only the `job` label will be left on the time series it gets a `job:` prefix, and we're summing so it's a `:sum` suffix.

In the global Prometheus config we pull in this timeseries:

```yaml
global:
  external_labels:
    datacenter: global  # In a HA setup, this would be global1 or global2

scrape_configs:
  - job_name: datacenter_federation
    honor_labels: true
    metrics_path: /federate
    params:
      match[]:
        - '{__name__=~"^job:.*"}'
    static_configs:
      - targets:
        - eu-west-1-prometheus:9090
```
etc.

The `match[]` here requests all job-level time series, so by following this naming convention you don't have to adjust the config every time there's a new aggregating rule.

Now you can use the expression `sum without(datacenter)(job:node_memory_MemTotal:sum)` to get the memory available across your entire global fleet!

The second general use case for federation is when you want to pull a handful of time series from a sibling Prometheus. For example as a sanity check, you may wish to check that your HAProxy Prometheus and your frontend Prometheus both are seeing about the same number of requests getting through.

The config would look similar to the above, though you'd ask for each time series you want explicitly by name. If the Prometheus is run by another team, don't forget to ask them first as reaching into someone else's Prometheus can be like reaching into the undocumented internals of a library in terms of maintenance and stability.

Using federation like this is usually only done for alerting, as Grafana supports multiple data sources (e.g. Prometheus servers) being used in one graph.

In both the above cases, federation is being used to pull in a limited and aggregated set of time series from another Prometheus. That Prometheus is otherwise continuing to do its duty, firing alerts and serving graph queries.

Where federation isn't suitable is if you use it to pull large swatches of time series - or even all time series - from another Prometheus, and then do alerting and graphing only from there. There's three broad reasons for this.

The first is performance and scaling. As the limiting factor of Prometheus performance is how much can be handled by a single machine, routing all your data to one global Prometheus limits your monitoring to what one machine can handle. By instead pulling only aggregated time series you're only limited to what one datacenter Prometheus can handle, allowing you to add new datacenters without having to worry about scaling the global Prometheus. The federation request itself can also be heavy to serve for the receiving Prometheus.

The second is reliability. If the data you need to do alerting is moved from one Prometheus to another then you've added an additional point of failure. This is particularly risky when WAN links such as the internet are involved. As far as is possible, you should try and push alerting as deep down the federation hierarchy as possible. For example an alert about a target being down should be setup on the Prometheus scraping that target, not a global Prometheus which could be several steps removed.

The third is correctness. Due to how it works, federation will pull in data some time after it was scraped and may also miss some data due to races. While the artifacts and oddness this causes in a global Prometheus are generally tolerable in a setup where your datacenter Prometheus servers are the primary workhorses for graphing and alerting, this is much more likely to cause issues when you're federating everything.

These issues don't just apply between datacenter and global Prometheus servers. Some have attempted to use a Prometheus as a type of proxy server, using federation to pull all data out of the scraping Prometheus into another Prometheus where all the real work is done. There are issues with this, such as the above mentioned correctness problems. If you find yourself in this situation either make the scraping Prometheus handle alerting and graphing, or do the scrapes via an actual proxy server using `proxy_url`.
