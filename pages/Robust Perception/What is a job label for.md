---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/what-is-a-job-label-for
author: [[Brian Brazil]] 
---
> The job label is one of the labels your targets will always have. So how can you use it?

# What is a job label for?


The `job` label is one of the labels your targets will always have. So how can you use it?

The `job` label is special as if after target relabelling if there is no `job` label present on a target, then the value of the `job_name` field  will be used. In this way your targets will always have a `job` label.

How best to use it then? What I would recommend is using the `job` label to organise applications which do the same thing, which almost always means processes running the same binary with the exact same configuration. For example you might have web frontends with `job="frontend"` and a Redis used as a cache with `job="redis"`. Using labels like this it's easy to aggregate across a job, for example CPU usage per job would be `sum by (job)(rate(process_cpu_seconds_total[5m]))`. As a counterexample a `job="kubernetes"` for all of your processes running on Kubernetes isn't particularly useful, you should aim for more meaningful `job` labels.

If you were running different sets of Redis servers for different purposes, it'd make sense to have different `job` labels for each set. Each set will likely have distinct performance characteristics after all, and aggregating them together would be unlikely to be of much use. If there are subdivisions within a set, such as sharding, it often makes sense to add a `shard` label or similar to subdivide the job.

Something to avoid is encoding information beyond that what sort of binary/configuration the process is in a `job` label. For example rather than having `job="redis_production"` it'd be better to have a separate `env="production"` to distinguish all of the production environment from development and staging environments. This makes it easier to do cross-environment analysis, without having to resort to regular expressions against label values.

_Wondering how to organise your scrape configs? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
