---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/pre-populated-silence-urls
author: [[Brian Brazil]] 
---
> You don't have to fill in all the silence details yourself.

# Pre-populated silence URLs


You don't have to fill in all the silence details yourself.

We've [previously](https://www.robustperception.io/pre-creating-alertmanager-silences) [looked](https://www.robustperception.io/creating-alertmanager-silences-from-python) at creating Alertmanager silences from the command line and scripts. When you're oncall though, you'll often be creating silences for firing alerts by hand to keep your pager quiet while you investigate. The usual way is not to manually type out all the silence details after clicking the New Silence button in the Alertmanager UI, but rather to click the Silence button for a relevant alert and then broaden the silence as needed.

Why broaden the silence? If for example a given alert is firing in one datacenter due to a software bug, it's likely going to soon start firing in all the other datacenters too. So you'd want the silence to not be tied to just the datacenter that happened to hit the alert threshold first.

Another thing you could do is use [notification templating](https://prometheus.io/docs/alerting/latest/notification_examples/) to include a link to the Alertmanager's New Silence form with the matchers pre-populated. The form is at `#/silences/new?filter` in the UI, and takes `filter` and `comment` parameters. For example `filter={alertname="ExampleAlertAlwaysFiring",job=~"p.*"}&comment=A comment` has equality and regex matchers, plus a comment. After URL encoding this would become [http://demo.robustperception.io:9093/#/silences/new?filter={alertname%3D%22ExampleAlertAlwaysFiring%22}&comment=Example%20comment](http://demo.robustperception.io:9093/#/silences/new?filter={alertname%3D%22ExampleAlertAlwaysFiring%22}&comment=Example%20comment) on the demo server.

This endpoint is not a formal part of the Alertmanager's API, rather it's an implementation detail of the UI. Accordingly it may change in the future without warning as the UI evolves.

_Want to reduce spammy notifications? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)