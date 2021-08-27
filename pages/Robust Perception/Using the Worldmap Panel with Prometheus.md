---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/using-the-worldmap-panel-with-prometheus
author: [[Brian Brazil]] 
---
> The Worldmap Panel for Grafana allows displaying of metrics on a map.

# Using the Worldmap Panel with Prometheus


The [Worldmap Panel](https://grafana.com/plugins/grafana-worldmap-panel) for Grafana allows displaying of metrics on a map.

Grafana has many [plugins](https://grafana.com/plugins), the one I'm going to look at today is the Worldmap Panel. First off install it in your Grafana if you don't have it already:

grafana-cli plugins install grafana-worldmap-panel

and then restart Grafana to pick it up.

Let's say you had a metric that looked like:

a\_metric{country="GB"} 4
a\_metric{country="IE"} 10
a\_metric{country="DE"} 15
a\_metric{country="FR"} 7

If you add a Worldmap panel in you dashboard you can then use this by setting the query to `a_metric`, setting the Legend Format to `{{country}}` so that the plugin will look at the country label, and also check the Instant checkbox. This will look like:

[![](https://www.robustperception.io/wp-content/uploads/2018/03/Screenshot_2018-03-29_17-44-04-640x360.png)](https://www.robustperception.io/wp-content/uploads/2018/03/Screenshot_2018-03-29_17-44-04.png)

In addition to [ISO 3166 two-letter codes](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2), you can also change the Location Data option on the Worldmap tab to support [ISO 3166 three-letter codes](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) and US states. It's also possible to provide a custom set of location labels [via a HTTP endpoint serving JSON](https://github.com/grafana/worldmap-panel#map-data-options).

You can of course use any PromQL expression here, so you could aggregate up usage by country for example.

_Wondering how to best visualise your data? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
