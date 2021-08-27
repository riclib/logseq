---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus
source: https://www.robustperception.io/openmetrics-is-released
author: [[Brian Brazil]]
---
> Coming soon to a standards body near you.

# OpenMetrics is released


Coming soon to a standards body near you.

OpenMetrics has been regularly worked on since June 2017. The goal is to take the Prometheus exposition text format which is a defacto standard, and make it into a cleaner vendor-neutral standard. As of earlier this month, the standard is [now available!](https://github.com/OpenObservability/OpenMetrics/blob/master/specification/OpenMetrics.md) The next steps are to bring it to the IETF with the goal of becoming an internet standard RFC.

While the OpenMetrics text format is very similar to the Prometheus text format, they are not the same thing. So how to tell the difference? The easiest way is to look for a counter, as the \_total convention has become a requirement with OpenMetrics. A counter in OpenMetrics looks like:

\# TYPE foo\_seconds counter
foo\_seconds**\_total** 1.0

Whereas in the Prometheus format the same counter would look like:

\# TYPE foo\_seconds**\_total** counter
foo\_seconds**\_total** 1.0

Notice that the `_total` suffix is not included in the `TYPE` in OpenMetrics.

This is the only major breaking change in the format. I'd recommend getting ahead of it and ensuring that all of your counters end in `_total` now, rather than being caught out when your client library is changed to use the OpenMetrics data model.

Beyond that, the upgrade to OpenMetrics should be pretty smooth. In fact, if you are using the Prometheus Python client and either Prometheus or DataDog you have probably been scraping using OpenMetrics for the past 2 years! You can safely use the new features of OpenMetrics in a client library even if the Prometheus text format is still being used on the wire. This is as we want the same samples to be ingested into Prometheus no matter which format is in use, to allow for transparent upgrades and downgrades. The way this is done is that the newer metrics are converted into their equivalents in the Prometheus text format, so for example the `_created` value that a counter can now have in OpenMetrics would end up as a separate gauge in the Prometheus text format.

There's been various other changes, warts removed, and some small features added. For example timestamps are now in seconds for consistency, and there's now only one form of escaping, rather than two. OpenMetrics requires an explicit `# EOF` at the end of the exposition, so that partial scrapes can be detected and treated as failed. Exemplars can be provided on histogram buckets and counters, allowing for trace id that can be linked to your tracing system or logs.

Info and StateSet have been added as types, formalising the existing practices for these in the Prometheus ecosystem. There's also now a GaugeHistogram type, which covers a rare use case which wasn't adequately covered by the existing Histogram type. There's also considerations to ensure that OpenMetrics works in a way that's suitable for both push and pull collection styles.

OpenMetrics will hopefully help vendors expose metrics in a standard format that all metrics monitoring systems can take advantage of, so that each vendor can focus on their monitoring system itself rather than having to integrate with every possible application out there.

_Need help planning for OpenMetrics? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
