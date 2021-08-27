---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/dont-try-to-swim-upstream
author: [[Brian Brazil]] 
---
> Have you ever felt that a piece of software just isn't doing what you need?

# Don’t Try to Swim Upstream


Have you ever felt that a piece of software just isn't doing what you need?

Software tends to be designed with a specific set of use cases in mind, with design principles to facilitate those use cases, and then when it comes to implementation tradeoffs are made based on those principles. While it's unlikely that everyone's use case will be perfectly met, if you go too far against the grain at some point you're just making life hard for yourself.

As an example [Prometheus counters cannot be set](https://www.robustperception.io/setting-a-prometheus-counter). You could try to hack around it by creating a new counter each time and incrementing it once, however if you use the APIs as intended with custom collectors and `MustNewConstMetric` your code will not only be simpler and but also less likely to have thread safety issues.

That's a relatively simple case. There's a myriad of different ways in you can in essence try to make Prometheus do push rather than pull, such as [via queues](https://www.robustperception.io/putting-queues-in-front-of-prometheus-for-reliability). Example reasons for wanting to do this vary from misunderstandings of reliability principles, to working around political issues within an organisation. Whatever the reason, fundamentally Prometheus is a pull system and if you try to make it do push instead (outside of the [intended use cases for the Pushgateway](https://prometheus.io/docs/practices/pushing/)) you're likely to find yourself running into an ever increasing level of problems.

While a software project could choose to add features to support use cases that are against its core design principles, longer term that would likely result in a design lacking cohesiveness and thus leaving all users less unsatisfied. For example a few years back the Prometheus developers discussed what'd happen if support for push was added, and our analysis was that we'd end up with a system with the combined disadvantages of both push and pull.

There are always going to be cases where you need to cheat a little to make things work, however if you find yourself systemically swimming upstream against the clearly stated philosophy of a given piece of software maybe you should step back and see if there's a simpler design. Or maybe even consider if you've maybe chosen the wrong piece of software for your needs.

_Unsure if Prometheus makes sense for your use case? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
