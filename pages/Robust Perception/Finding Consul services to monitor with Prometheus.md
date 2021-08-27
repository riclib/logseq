---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/finding-consul-services-to-monitor-with-prometheus
author: [[Brian Brazil]] 
---
# Finding Consul services to monitor with Prometheus

> One of the service discovery methods Prometheus supports is Consul. Let's look at how to use it.

---
One of the service discovery methods Prometheus supports is [Consul](https://prometheus.io/docs/operating/configuration/#%3Cconsul_sd_config%3E). Let's look at how to use it.

Finding targets happens in two stages. First a service discovery method such as Consul returns potential targets with metadata. Secondly relabelling allows you to choose which of those targets you want to scrape, and how to convert the metadata into target labels. We've looked at the [theory of relabelling](http://www.robustperception.io/life-of-a-label/) previously, so let's make this more concrete.

Lets say you wanted to monitor all services with a `prod` tag and use the Consul service name as the job label. Your scrape configuration would look like:
```yaml

scrape_configs:
  - job_name: dummy
    consul_sd_configs:
      - server: 'localhost:8500'
    relabel_configs:
      - source_labels: [__meta_consul_tags]
        regex: .*,prod,.*
        action: keep
      - source_labels: [__meta_consul_service]
        target_label: job
```
The first relabel action says to keep processing only those targets who have a `prod` tag. Prometheus exposes the Consul tags as a comma separated list in the label called `__meta_consul_tags`, with leading and trailing commas added for [convenience](http://www.robustperception.io/little-things-matter/).

The second relabel action says to copy the service name from the `__meta_consul_service` label to the `job` label. This is take advantage of the default values for relabel actions, as a straight copy from one label to another is common.

What if you wanted to do something more complicated?

Let's say you wanted the job label to come from a tag with the format `job-XXX`. We could replace the last relabel action with:
```yaml

      - source_labels: [__meta_consul_tags]
        regex: .*,job-([^,]+),.*
        replacement: '${1}'
        target_label: job
```
This extracts the part of the Consul tag that's of interest to us, and copies it to the `job` label.  `([^,]+)` means to capture at least one character which is not a comma.

This is just a small taste of what's possible. With the flexibility of relabelling you can map however you think about your services into Prometheus labels!
