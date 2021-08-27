---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/common-pitfalls-when-using-the-pushgateway
author: [[Brian Brazil]] 
---
> Jobs of an ephemeral nature are often not around long enough to have their metrics scraped by Prometheus. In order to remedy this the Pushgateway was developed to allow for these types of jobs to push their metrics to a metrics cache in order to be scraped by Prometheus long after the original jobs have gone away. This blogpost discusses some of the common pitfalls users tend to fall into when adding the Pushgateway to their monitoring stack.

# Common pitfalls when using the Pushgateway


Jobs of an ephemeral nature are often not around long enough to have their metrics scraped by Prometheus. In order to remedy this the Pushgateway was developed to allow for these types of jobs to push their metrics to a metrics cache in order to be scraped by Prometheus long after the original jobs have gone away. This blogpost discusses some of the common pitfalls users tend to fall into when adding the Pushgateway to their monitoring stack.

In the majority of cases, the recommended approach to monitoring something with Prometheus is to stick to the normal pull model, however there are a small number of cases where the Pushgateway is preferable. In most cases, the only valid use case for the Pushgateway is in collecting metrics from service-level batch jobs.

One common pitfall of using the Pushgateway with Prometheus is that it becomes a single point of failure. If your Pushgateway is collecting metrics from many different sources and goes down, you will lose monitoring of all of those sources, potentially triggering a lot of needless alerts.

Another important point to remember is that the Pushgateway will not automatically remove any metrics pushed to it. This means that metrics whose source may disappear will not disappear from Prometheus scraping the Pushgateway. This is particularly evident with metrics containing an `instance` label (which should not be going to the Pushgateway in the first place, as they are not service-level). Instances may come and go but the old metrics for the expired instances will remain in the Pushgateway and thus Prometheus. In order to synchronize, one must remember to delete expired metrics from the Pushgateway using its API:

curl -X DELETE http://pushgateway.example.org:9091/metrics/job/some\_job/instance/some\_instance

Another typical misuse of the Pushgateway includes efforts to circumvent firewall or NAT issues preventing Prometheus from scraping its desired targets. Rather than use Pushgateway to push metrics to for Prometheus to scrape, the recommended approach would be to move Prometheus behind that firewall, closer to the targets we want to scrape.  
For getting around NAT, you can try Robust Perception's own [PushProx](https://github.com/robustperception/pushprox).

Finally, remember that when using Pushgateway, you lose the `up` metric.

_Want more Prometheus related best practices? [Contact us](mailto:prometheus@robustperception.io)._

Published by Conor Broderick in [Posts](https://www.robustperception.io/category/posts)
