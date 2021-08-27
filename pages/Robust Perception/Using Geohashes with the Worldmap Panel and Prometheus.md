---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/using-geohashes-with-the-worldmap-panel-and-prometheus
author: [[Brian Brazil]] 
---
> Wouldn't it be nice to have arbitrary locations on the Worldmap panel?

# Using Geohashes with the Worldmap Panel and Prometheus


Wouldn't it be nice to have arbitrary locations on the Worldmap panel?

In the [previous post](https://www.robustperception.io/using-the-worldmap-panel-with-prometheus/) we looked at using the Worldmap panel with the pre-set countries and US states. If you wanted locations beyond that you can provide them via a [single JSON endpoint](https://github.com/grafana/worldmap-panel#map-data-options), however there is also a way to have the location come from the time series themselves. The `geohash` location data option is targeted at data from Elasticsearch, but is also usable for Prometheus.

A geohash is a way of representing a 2-d rectangle based on longitude and latitude. There are websites online where you can [pick them off a map](http://geohash.gofreerange.com/) or [convert from other formats](http://www.movable-type.co.uk/scripts/geohash.html).

Once you have your geohash you can put it in a metric, Dublin for example would be:

a\_metric{geohash="gc7x",place="Dublin"} 4

From there you can edit your Worldmap panel in Grafana, on the Metrics tab setting the query to `a_metric`, Format as to Table, and enabling Instant Query. On the Worldmap tab, set Location Data to geohash, ES Metric Field must be set to `Value`, our geohash is in the `geohash` label so set ES geo\_point Field to `geohash`, and here the label of the location is in the `place` label, so set ES Location Name Field to `place`:

[![](https://www.robustperception.io/wp-content/uploads/2018/04/Screenshot_2018-04-06_14-33-23.png)](https://www.robustperception.io/wp-content/uploads/2018/04/Screenshot_2018-04-06_14-33-23.png)

The end result would look like:

[![](https://www.robustperception.io/wp-content/uploads/2018/04/Screenshot_2018-04-06_14-33-11-640x372.png)](https://www.robustperception.io/wp-content/uploads/2018/04/Screenshot_2018-04-06_14-33-11.png)

You could use this for example to have each of your datacenters report a metric with its location, federate that metric to a global Prometheus, and use that for a global dashboard. As Prometheus does not have any understanding of geohashes, any aggregation across different geohashes would need to be done before the metric was exposed.

_Wondering how to best visualise your global data? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
