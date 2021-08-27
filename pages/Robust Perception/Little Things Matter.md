---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/little-things-matter
author: [[Brian Brazil]] 
---
# Little Things Matter

> As part of designing and building Prometheus, hundreds of technical decisions have to be made. Every one of them is important in building a sustainable consistent ecosystem. Today, let's look at one small decision that was made by the Prometheus developers in Consul service discovery.

---
As part of designing and building [Prometheus](https://prometheus.io/), hundreds of technical decisions have to be made. Every one of them is important in building a sustainable consistent ecosystem. Today, let's look at one small decision that was made by the Prometheus developers in Consul service discovery.

Consul has the idea of tags, a list of strings that can be associated with a service. Prometheus has the idea of labels, a set of key-value pairs. How do we allow users to use Consul tags in Prometheus?

The answer is that we take the list of tags and put them all in one label called `__meta_consul_tags`. As semicolon is the default separator used when combining multiple labels, commas are used for joining tags. This little detail means it's easy to distinguish different labels from different tags.

So say we have services with tags `tagA`, `tagB` and `tagC` which would become `tagA,tagB,tagC`. How would you scrape only services that had `tagB`?
```yaml
scrape_configs:
  - job_name: example
    consul_sd_configs:
      - server: 'localhost:8500'
    relabel_configs:
      - source_labels: [__meta_consul_tags]
        regex: .*,tagB,.*
        action: keep
```

Those who are used to using with regexes will spot a problem with this, what if `tagB` is the first or last tag? You'd have use a regex along the lines of `(^|.*,)tagB(,.*|$)` to allow for this.

That's kind of annoying, and not everyone would spot that it was required for correctness. So we decided to make a little change. Now we always put a comma before and after any tags, so that the above configuration always works as expected.

This is a little detail, but it avoids confusion and reduces the number of things our users need to think about. Since then this pattern has been copied into other service discovery mechanisms such as EC2 where we also have to convert lists into a string.

When designing your interfaces, look out for little potential gotchas and see if you can tweak things so that they don't trip up your users.

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
