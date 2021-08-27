---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/laying-out-alertmanager-routes
author: [[Brian Brazil]] 
---
> How should you design your Alertmanager routes for flexibility and growth?

# Laying out Alertmanager routes


How should you design your Alertmanager routes for flexibility and growth?

Starting out with a small setup for one-ish team, it's fine to wing it in terms of your Alertmanager routes. You'll only have a handful of simple business logic you want, such as pages go to oncall, everything else goes to a ticketing system. There's not much need for engineering when you've only basic requirements.

In a larger organisation though, things aren't so basic. You probably have many many teams, each with their own needs in terms of where they want their notifications to go. It's usual that an organisation will share a single set of Alertmanagers, so the first thing to do is establish some standard label such as `service` or `team` as an `external_label` on every Prometheus to signify what team it is alerting for. In the less common case where a Prometheus is alerting for multiple teams, this would instead be a label on the alerting rule.

From there have a route per team under the root route, using the distinguishing label to match each team's alerts. Each team can then own their own alerts within their route, without affecting other teams. Each team can then have subroutes that might for example treat non-production pages as tickets, send pages on to the oncall, and everything else as tickets. Over time there may be more routes to cover subtleties in the service, such as applying different `group_by` or `group_interval` for some notifications.

If each team doesn't need to be able to fully customise their notification handling, you could also [use labels to direct notifications](https://www.robustperception.io/using-labels-to-direct-email-notifications). This tends to make sense when teams are "customers" of some centrally managed service, rather than running a service themselves. You could also end up mixing and matching, depending on how services are run in your organisation.

Alertmanager configuration should change rarely for a given team, every few months at most usually, as while alerting rules may change often that's not the case for how you want to route alerts. Accordingly keeping everything in one big file with any changes being done via tickets or PRs to the monitoring team who runs the alertmanagers is often practical. For something a bit more self service you could give each team a file in source control that they can edit with their routes and receivers, and pull that all together automatically. YAML is machine readable and writable after all. Don't forget precommit checks or similar to catch accidental breakage, at the least that the Alertmanager can parse the resultant configuration. One other thing to be aware of is that inhibition rules are currently global, so be wary of allowing teams to create them as they may inadvertently inhibit other teams' alerts.

If there's any other organisation-wide routing you'd like to do, put it ahead of the per-team routes. For example some organisations like to log all alerts via a webhook and `continue` route. You should also ensure that the root or fallback route has a reasonable destination, so that if an alert isn't caught by another route you can follow up on it. For example you might have such alerts go to tickets once a day.

So that's how to handle it. Use labels to distinguish which team owns what alerts, and then give them control of a routing subtree for their alerts.

_Unsure if your routing tree is reasonable? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
