---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/scaling-and-federating-prometheus
author: [[Brian Brazil]] 
---
> A single Prometheus server can easily handle millions of time series. That's enough for a thousand servers with a thousand time series each scraped every 10 seconds. As your systems scale beyond that, Prometheus can scale too.

# Scaling and Federating Prometheus


A single [Prometheus](https://prometheus.io/) server can easily handle millions of time series. That's enough for a thousand servers with a thousand time series each scraped every 10 seconds. As your systems scale beyond that, Prometheus can scale too.

## Initial Deployment

When starting out it's best to keep things simple. A single Prometheus server per datacenter or similar failure domain (e.g. EC2 region) can typically handle a thousand servers, so should last you for a good while. Running one per datacenter avoids having the internet or WAN links on the critical path of your monitoring.

If you've more than one datacenter, you may wish to have global aggregates of some time series. This is done with a "global Prometheus" server, which federates from the datacenter Prometheus servers.

\- scrape\_config:
  - job\_name: dc\_prometheus
    honor\_labels: true
    metrics\_path: /federate
    params:
      match\[\]:
        - '{\_\_name\_\_=~"^job:.\*"}'   # Request all job-level time series
    static\_configs:
      - targets:
        - dc1-prometheus:9090
        - dc2-prometheus:9090

It's suggested to run two global [Prometheis](http://prometheus.io/docs/introduction/faq/#what-is-the-plural-of-prometheus) in different datacenters. This keeps your global monitoring working even if one datacenter has an outage.

## Splitting By Use

As you grow you'll come to a point where a single Prometheus isn't quite enough. The next step is to run multiple Prometheus servers per datacenter. Each one will own monitoring for some team or slice of the stack. A first pass may result in fronted, backend and machines (node exporter) for example.

As you continue to grow, this process can be repeated. MySQL and Cassandra monitoring may end up with their own Prometheis, or each Cassandra cluster may have a Prometheus server dedicated to it.

You may also wish to start splitting by use before there are performance issues, as teams may not want to share Prometheis or to improve isolation.

## Horizontal Sharding

When you can't subdivide Prometheus servers any longer, the final step in scaling is to scale out. This usually requires that a single job has thousands of instances, a scale that most users never reach. This is more complex setup and is much more involved to manage than a normal Prometheus deployment, so should be avoided for as long as you can.

The architecture is to have multiple sharded Prometheis, each scraping a subset of the targets and aggregating them up within the shard. A leader federates the aggregates produced by the shards, and then the leader aggregates them up to the job level.

On the shards you can use a hash of the address to select only some targets to scrape:

global:
  external\_labels:
    shard: 1  # This is the 2nd shard. This prevents clashes between shards.
scrape\_configs:
  - job\_name: some\_job
    # Add usual service discovery here, such as static\_configs
    relabel\_configs:
    - source\_labels: \[\_\_address\_\_\]
      modulus:       4    # 4 shards
      target\_label:  \_\_tmp\_hash
      action:        hashmod
    - source\_labels: \[\_\_tmp\_hash\]
      regex:         ^1$  # This is the 2nd shard
      action:        keep

And the leader federates from the shards:

\- scrape\_config:
  - job\_name: shards
    honor\_labels: true
    metrics\_path: /federate
    params:
      match\[\]:
        - '{\_\_name\_\_=~"^shard:.\*"}'   # Request all shard-level time series
    static\_configs:
      - targets:
        - shard0:9090
        - shard1:9090
        - shard3:9090
        - shard4:9090

Information for dashboards is usually taken from the leader. If you wanted to drill down to a particular target, you'd do so via its shard.

_Have questions about scaling Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
