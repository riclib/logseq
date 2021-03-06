---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/translating-between-monitoring-languages
author: [[Brian Brazil]] 
---
# Translating between monitoring languages

> There's so many monitoring systems out there these days that it's difficult to figure out what's actually different, and what just has a different name or falls under a different concept. Let's look at the Graphite, InfluxDB and Prometheus query languages and see how the same ideas are represented in each.

---
There's so many monitoring systems out there these days that it's difficult to figure out what's actually different, and what just has a different name or falls under a different concept. Let's look at the [Graphite](http://graphite.readthedocs.io/en/latest/functions.html), [InfluxDB](https://docs.influxdata.com/influxdb/v1.2/query_language/spec/) and [Prometheus](https://prometheus.io/docs/querying/basics/) query languages and see how the same ideas are represented in each.

I'm going to approach this by showing how various expressions look in each of the languages, seeing where equivalents exist and where they don't. As each of these systems has slightly different evaluation models, design goals and features, I'm going to aim for expressions that do basically the same thing rather than expressions that produce exactly the same result.

Another difference is how information around expressions is specified. For example query start and end time are URL parameters for Graphite and Prometheus (`from`&`until` and `start`&`end` respectively), whereas from InfluxDB they're part of the expression (a `WHERE` clause on `time`). To keep things as concise and as apples-to-apples as possible, these will not be included in the examples. The various graphing related functions in Graphite such as [color](http://graphite.readthedocs.io/en/latest/functions.html#graphite.render.functions.color) and [stacked](http://graphite.readthedocs.io/en/latest/functions.html#graphite.render.functions.stacked) will similarly be ignored.

In terms of syntax, Graphite uses nested prefix functions. InfluxQL is SQL-like. PromQL has a C expression-like language with added modifiers.

I'm working primarily off documentation and examples provided by the projects, so please excuse any subtleties or features I missed. That said, let's get going.

## Math

The first category is arithmetic and functions that work independently on data points. Let's start off with adding 1 to all values across a set of time series:
```
Graphite: offset(seriesname.*, 1)
InfluxQL: SELECT 1 + "value" FROM "seriesname"
PromQL: seriesname + 1
```
Graphite has various other functions such as `invert`, `scale` and `pow` for the other basic arithmetic operators. [InfluxQL](https://docs.influxdata.com/influxdb/v1.2/query_language/math_operators/) and [PromQL](https://prometheus.io/docs/querying/operators/#arithmetic-binary-operators) offer the basics of +, -, / and *. Prometheus in addition offers % and ^, and allows full expressions on both sides of the operators. None of the languages offer bitwise operators.

What about functions?
```
Graphite: logarithm(seriesname.*, 1)
InfluxQL: Not supported.
PromQL: log10(seriesname)
```
Graphite alone has `sin`, and PromQL alone offers various [functions](https://prometheus.io/docs/querying/functions/) like `ceiling`, `clamp_min` and `day_of_month`.

I'm a bit surprised to find that InfluxQL has no math functions, there's only TODOs for [ceiling](https://docs.influxdata.com/influxdb/v1.2/query_language/functions/#ceiling) and [floor](https://docs.influxdata.com/influxdb/v1.2/query_language/functions/#floor). I will however note that TICKscript's lambda appears to support all the [functions](https://docs.influxdata.com/kapacitor/v1.2/tick/expr/#math-functions) provided by [Go](https://golang.org/pkg/math/), if you're using [Kapacitor](https://github.com/influxdata/kapacitor) in addition to InfluxDB. As a stream processor there isn't a query interface though. It's more like Prometheus's [recording rules](https://prometheus.io/docs/querying/rules/).

As you can see, which functions are supported varies quite a bit. The usefulness of the functions also varies, do you really need a cosine when doing services monitoring? In principle it's not hard to add more if you need them though, all three systems are actively maintained.

## Selecting Series

You don't always want every metric. Often there's some level of filtering desired, whether it be strict lookup:
```
Graphite: seriesname.filter
InfluxQL: SELECT "value" FROM "seriesname" WHERE tag="filter"
PromQL: seriesname{labelname="filter"}
```
Or a regular expression:
```
Graphite: grep(seriesname, "regexp")
InfluxQL: SELECT "value" FROM "seriesname" WHERE tag =~ /regexp/
PromQL: seriesname{labelname~="regexp"}
```
Positive and negative matches are supported by all three languages.

## Working Across Series

Often you'll want to take a time slice, and do some form of aggregation on the data points in that slice. Let's average the points:
```
Graphite: averageSeries(seriesname.*)
InfluxQL: SELECT MEAN("value") FROM "seriesname"
PromQL: avg(seriesname)
```
These are called [aggregation operators](https://prometheus.io/docs/querying/operators/#aggregation-operators) in PromQL, Graphite and InfluxQL have similar functions to do the same for counts, sums, standard deviations, quantiles and top/bottom K series.

A notable difference is that the Graphite functions for top/bottom K work across both series and time, whereas the PromQL equivalents operate independently at each point in time. This often causes confusion, as `top(seriesname, 5`) on a Prometheus graph can return more than 5 time series. What you can do is make one PromQL query to determine the series you want, and another to then graph them.

There's also a few additional notable functions InfluxQL/Graphite have:

# Range.
```
Graphite: rangeOfSeries(seriesname.*)
InfluxQL: SELECT SPREAD("value") FROM "seriesname"
PromQL: max(seriesname) - min(seriesanme)
```

# Number of unique values.
```
Graphite: Not supported.
InfluxQL: SELECT DISTINCT("value") FROM "seriesname"
PromQL: count(count_values(seriesname))
```

# Mode.
```
Graphite: Not supported.
InfluxQL: SELECT MODE("value") FROM "seriesname"
PromQL: topk(count_values(seriesname)), 1)
```

Per the above you can see that PromQL has a `count_values`, I believe this will be the [histogram](https://docs.influxdata.com/influxdb/v1.2/query_language/functions/#histogram) function in InfluxQL once it is implemented. Graphite has no equivalent.

It would be remiss of me not to include an example of InfluxQL [subqueries](https://docs.influxdata.com/influxdb/v1.2/query_language/data_exploration/#subqueries):

```
Graphite: averageAbove(water.level.h2o.feet.*, 5)
InfluxQL: SELECT "all_the_means" FROM (SELECT MEAN("water_level") AS "all_the_means" FROM "h2o_feet" GROUP BY time(12m)) WHERE "all_the_means" > 5
PromQL: avg_over_time(h20_water_level_feet[12m]) > 5
```

There's one final function in Prometheus that technically fits in this section, which the others don't support and that is [histogram_quantile](https://prometheus.io/docs/querying/functions/#histogram_quantile). It allows histogram data to be correctly aggregated and processed to produce a quantile.

## Working Across Time

Let's go the other direction now. Working on the history of each of a set of time series. Once again let's use an average:

```
Graphite: movingAverage(seriesname.*, '1hour')
InfluxQL: SELECT MOVING_AVERAGE(MEAN("value"), 1) FROM "seriesname" GROUP_BY time(60m)
PromQL: avg_over_time(seriesname[1h])
```

In Graphite similar functions exist for [quantiles](http://graphite.readthedocs.io/en/latest/functions.html#graphite.render.functions.nPercentile) and [sums](http://graphite.readthedocs.io/en/latest/functions.html#graphite.render.functions.movingSum), and InfluxQL's `[MOVING_AVERAGE](https://docs.influxdata.com/influxdb/v1.2/query_language/functions/#moving-average)` also supports counts, min and max on top of that. Prometheus adds [standard deviation](https://prometheus.io/docs/querying/functions/#aggregation-_over_time) on top of that again.

In addition Graphite has [`summarize`](http://graphite.readthedocs.io/en/latest/functions.html#graphite.render.functions.summarize), which is `movingSum` with non-overlapping data. PromQL's `sum_over_time` can fill both roles, and InfluxQL's `GROUP_BY time()` can do similar.

All three systems have ways of dealing with counters, which is to say series that only go up and also sometimes reset. Graphite and InfluxQL offer `nonNegativeDerivative`, which throws away the data point at reset. PromQL offers `rate()`, which directly handles the reset and makes allowances for various failure modes.

For gauges that can go up and down there's [`linearRegressionAnalysis`](http://graphite.readthedocs.io/en/latest/functions.html#graphite.render.functions.linearRegressionAnalysis)/`[derivative](http://graphite.readthedocs.io/en/latest/functions.html#graphite.render.functions.derivative)` in Graphite, [derivative](https://docs.influxdata.com/influxdb/v1.2/query_language/functions/#derivative)/[difference](https://docs.influxdata.com/influxdb/v1.2/query_language/functions/#difference) in InfluxQL, and `[deriv](https://prometheus.io/docs/querying/functions/#deriv())`/[`idelta`](https://prometheus.io/docs/querying/functions/#idelta()) in PromQL.

Mildly related is the ability to move data around in time. Graphite has [timeShift](http://graphite.readthedocs.io/en/latest/functions.html#graphite.render.functions.timeShift), Prometheus has the `[offset](https://prometheus.io/docs/querying/basics/#offset-modifier)` modifier. InfluxQL has a [github issue](https://github.com/influxdata/influxdb/issues/5930) for lag/lead.

Here's another example of a InfluxQL subquery, allowing a sum on top of a max:

```
Graphite: sumSeries(summarize(water.level.h2o.feet.*, '1hour', 'max'))
InfluxQL: SELECT SUM("max") FROM (SELECT MAX("water_level") FROM "h2o_feet" GROUP BY "location")
PromQL: sum(max_over_time(water_level[1h]))
```

Finally we'll look at predictions and smoothing (beyond the averages above). PromQL offers [Holt-Winters](https://prometheus.io/docs/querying/functions/#holt_winters()) smoothing and [least-squares prediction](https://prometheus.io/docs/querying/functions/#predict_linear). [Graphite](http://graphite.readthedocs.io/en/latest/functions.html#graphite.render.functions.holtWintersForecast) and [InfluxQL](https://docs.influxdata.com/influxdb/v1.2/query_language/functions/#holt-winters) offer Holt-Winters prediction.

## Binary Operators

This is where the fun starts. What if you want to do math between two different time series? Let's start with simple subtraction:

```
Graphite: reduceSeries(my.*, "diffSeries", 1, "a", "b")
InfluxQL fields: SELECT "a" - "b" FROM "table"
InfluxQL measurements: Not supported.
PromQL: my_a - my_b
```

While Influx doesn't support [math across measurements](https://github.com/influxdata/influxdb/issues/3552) (which would be a common need), once again TICKscript [does](https://docs.influxdata.com/kapacitor/v1.2/nodes/join_node/):

```
var a = stream
    | from()
        .measurement('a')
var b = stream
    | from()
        .measurement('b')
a
    | join(b)
        .as('a', 'b')
        .tolerance(1s)
    | eval(lambda: "a.value" / "b.value")
```

Let's step up our game a bit. What if we want the proportion of the total that each of a set of series represents:

```
Graphite: asPercent(seriesname.*)
InfluxQL: Not supported (can probably be done with TICKscript's [Join.On](https://docs.influxdata.com/kapacitor/v1.2/nodes/join_node/#on)).
PromQL: seriesname / on() group_left sum(seriesname)
```

This is a rare time when Graphite is notably more succinct than PromQL for a relatively common operation.

Function, operators and aggregators can be composed in PromQL, so let's return all instances more than 2 standard deviations above the average ignoring which instance it is but preserving all other label structure:

```
Graphite: Not supported.
InfluxQL: Not supported (doesn't appear possible with TICKscript).
PromQL: 
    seriesname
  > without(instance) group_left
    2 * stddev ignoring (instance) (seriesname) + avg ignoring (instance) (seriesname)
```

How about which replica is the most behind its leader, ignoring the instance and job labels?

```
Graphite: Not supported.
InfluxQL: Not supported (doesn't appear possible with TICKscript).
PromQL: 
  topk by (cluster) (
      leader_position 
    - without(instance, job) group_right
      replica_position
  , 1)
```

Here the real power of PromQL's label handling becomes apparent. PromQL is designed to allow have configurable matching, grouping and propagation of labels.

## Outcomes

If you want to calculate one number at a time, Graphite, InfluxDB, and Prometheus have broadly the same functionality and it's usually possible to convert expressions between them.

If you're trying to choose one of them there's a few things to note. On the down side is InfluxQL's inability to do math across measurements and lack of math functions. Prometheus's execution model causing surprise for topK graphs is also worth remarking on, though that can be worked around.

On the up side, Graphite is more powerful than I had expected. With `histogram_quantile`, PromQL is the only language to offer support for aggregate quantiles on events.

The big win though is PromQL's ability to do grouping, slicing, dicing and processing based on labels. (Prometheus is also [Turing Complete](https://www.robustperception.io/conways-life-in-prometheus/), but don't try that in production).
