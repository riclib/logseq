---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/regex-selectors-are-a-smell
author: [[Brian Brazil]] 
---
> Have you ever found yourself having to keep on updating and tweaking certain regexes in PromQL?

# Regex Selectors are a Smell


Have you ever found yourself having to keep on updating and tweaking certain regexes in PromQL?

A smell doesn't automatically mean that something is bad. A smell is a sign however that you might want to take a closer look, that there might be a better way to do things.

Let's say you saw a PromQL expression that included the selector `a_metric{job=~".*apache.*"}`. Doesn't that look a bit odd? What's happening here is that labels are intended to be opaque strings, but a regex is being used to extract semi-structured information from it in PromQL. What happens if there's a new job that is a sidecar to Apache which also has this metric? It's presumably also contain the substring `apache` in the `job` label, and could be incorrectly picked up by this expression.

Maintaining this could get tricky over time, as you end up with either implicit or explicit rules about how this label looks like. If someone violates the rules, things get even more complicated. In addition regex selectors like these are less efficient, as for every query the regex has to be tested against every known label value.

So what's a better approach? Here I'd say that the `job` label should probably just be `apache` to indicate [applications doing the same thing](https://www.robustperception.io/what-is-a-job-label-for), and encode any other relevant information into other target labels using relabelling. For example rather than having `{job="myteam-apache-prod-hostname"}` for a target you could have `{job="apache",owner="myteam",env="prod",instance="hostname"}`.

Not everything [belongs as a target label](https://www.robustperception.io/target-labels-are-for-life-not-just-for-christmas), so some information such as when a job was launched and by who may be better represented by an info metric either from the application itself or using something along the lines of kube-state-metrics. This information is then available for the queries that need it, without making every single query touching a job's metric have to care about this extra information.

Target labels are only one place that you can have regexes. Another is instrumentation labels, the most common example being HTTP status codes. A selector like `http_requests_total{code=~"5.."}` as part of tracking errors is not uncommon. As long as the cardinality is bounded, the series are always present, and there's no need to for example ignore certain 5xx codes for certain applications this can be an okay tradeoff.

If however you find yourself encoding nuanced business logic about what is and isn't a failure into a regex selector, that's probably something that better lives inside the application itself rather than in your monitoring configuration. In simple cases you might for example add a metric called `myapp_http_requests_failed_total` which is incremented based on business logic inside the application, which can be maintained by the developers of the application as they're adding features. This also helps avoid needing to change the monitoring configuration every time a new version of the application is deployed.

At the end of the day a regex selector may be a perfectly fine tradeoff. It's worth giving some brief thought though to see if there might be a different approach that would make your life easier, even if it's not worth the effort of changing it right now

_Have a question about relabelling? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
