---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/extracting-full-labels-from-consul-tags
author: [[Brian Brazil]] 
---
# Extracting full labels from Consul tags

> Prometheus 1.3.0 contained a small change that makes it possible to extract arbitrary labels from systems like Consul that only normally support one-dimensional tags.

---
[Prometheus 1.3.0](https://www.robustperception.io/new-features-in-prometheus-1-3-0/) contained a small change that makes it possible to extract arbitrary labels from systems like Consul that only normally support one-dimensional tags.

Consul supports tags as a array of strings, compared to the key/value pairs that Prometheus labels allow. This usually results in tags such as "dev" and "prod", which require configuration on the Prometheus side to know that "env" is the associated label name:
```yaml
relabel_configs:
  - source_labels: [__meta_consul_tags]
    regex: '.*,(dev|prod),.*'
    replacement: '${1}'
    target_label: env
```
This is fine for a handful of labels and values. What if you wanted to give your users freedom to add arbitrary labels, without having to configure each one on the Prometheus side?

The answer is that as of 1.3.0 the target label for relabelling can also use regex substitution, so we can do:
```yaml
relabel_configs:
  - source_labels: [__meta_consul_tags]
    regex: '.*,([^=]+)=([^,]+),.*'
    replacement: '${2}'
    target_label: '${1}'
```
This will take a tag such as `a=b` and convert it to a label named `a` with the value `b`.

This can only extract one such label, using multiple actions permits many labels to be extracted:
```yaml
relabel_configs:
  - source_labels: [__meta_consul_tags]
    regex: ',(?:[^,]+,){0}([^=]+)=([^,]+),.*'
    replacement: '${2}'
    target_label: '${1}'
  - source_labels: [__meta_consul_tags]
    regex: ',(?:[^,]+,){1}([^=]+)=([^,]+),.*'
    replacement: '${2}'
    target_label: '${1}'
  - source_labels: [__meta_consul_tags]
    regex: ',(?:[^,]+,){2}([^=]+)=([^,]+),.*'
    replacement: '${2}'
    target_label: '${1}'
  - etc. etc.
```
This approach requires as many actions as potential tags. This would be best generated using a templating system.

This technique is not limited to Consul tags, the same will work for any other comma separated list of strings from service discovery.
