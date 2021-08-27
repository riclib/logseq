---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/remote-read-and-partial-failures
author: [[Brian Brazil]] 
---
> What happens when your clustered storage fails?

# Remote read and partial failures


What happens when your clustered storage fails?

A key design principle of Prometheus is reliability. Part of this is that it doesn't have any [hard dependencies on clustered systems](https://www.robustperception.io/monitoring-without-consensus), such as on Zookeeper, Kafka, or Cassandra that may have issues if there's fun like a network partition. As long as Prometheus has network access to get to a working Alertmanager it will keep on sending alerts.

This is great for resilience, but limits you to the amount of storage you can fit on one machine. That's a surprisingly large number on modern machines, but it is still a limitation. Remote read is a way to access data that you couldn't fit on on machine, but with safeties so that if the clustered storage system goes down that Prometheus can still evaluate PromQL expressions to send alerts, and produce graphs. In particular if you do an `query_range` API call which involves a failed request to remote read then the PromQL query will work off the local data within Prometheus and include a warning in the response. Even without the warning, having no results in your graphs before a certain time should make it pretty clear what's going on. So it's still a failure, but one you can easily reason about.

What happens however if the remote endpoint isn't hard down, but instead only answers half of requests? Taking a simple example say your expression was `foo unless bar` and `bar` failed, you would now return results that shouldn't be there. Conversely with `foo and bar` if `bar` failed you would fail to return results that should be there - and similar would happen with other binary operators. Trying to write expressions that are resilient to an arbitrary subset of your data being missing is not tractable. So what to do?

This is not the only place in Prometheus where partial data comes up, another is in scraping. If there is a partial scrape (e.g. a parse error is hit half way through, or `sample_limit` is hit) then the whole scrape is considered as failed, none of its samples are ingested, and `up` is set to 0. A scrape completely failing is already something you need to allow for, so it doesn't making writing alerts and graphs more difficult to handle the partial failure case in the same manner.

Remote read takes a similar approach. If one of the remote read requests to a remote read endpoint fails then Prometheus will treat it as though all of those requests to that endpoint failed. The remote read endpoint being hard down is already something you needed to allow for, so this doesn't add additional complexity in handling a partial failure. For example if you were going to surface the warning from a hard failure on a dashboard, that will automatically happen with a partial failure too.

Taking complex failures and making them behave like simpler more common failures makes life easier.

_Wondering how to scale your monitoring? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
