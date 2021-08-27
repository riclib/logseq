---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus
source: https://www.robustperception.io/conways-life-in-prometheus
author: [[Brian Brazil]]
---
> Some monitoring systems are very limited in what calculations you can do with them. Prometheus is not such a system, and today I'm happy to say that half a year after it publicly launched, Prometheus is Turing Complete.

# Conway’s Life in Prometheus


Some monitoring systems are very limited in what calculations you can do with them. Prometheus is not such a system, and today I'm happy to say that half a year after it publicly launched, Prometheus is Turing Complete.

[![Conway's Life](http://www.robustperception.io/wp-content/uploads/2015/08/Screenshot-180815-225102-e1439934834722.png)](http://www.robustperception.io/wp-content/uploads/2015/08/Screenshot-180815-225102.png)

One state of Conway's Life

A key feature of [Prometheus](https://prometheus.io/) is the query language that's called promql. This supports calculations including aggregations, predictions, various math functions and joins between time series. Another key feature is labels, key-value pairs associated with every time series. Together they allow processing and combining of many time series in parallel.

The most recently added function is `label_replace`, which does regular expressions replacements on labels. While regexp replacement has been a big part of how generic service discovery is done, it's the first promql function to allow direct manipulation of labels. This will form the core of my approach.

There's a few ways I could demonstrate this, I could implement a Turing Machine (as I did in an certain monitoring language in 2007), or an interpreter for a small language. In order to have something visual, I'm going to implement [Conway's Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life).

## Basics

Conway's Life is relatively simple. You have a rectangular grid. On each step of the algorithm a cell in the grid is alive if either it has exactly three alive neighbours, or it's already alive and has two live neighbours. To represent the grid we'll have a metric called `grid`. It'll have two labels, `x` and `y` with values `1`, `11`. `111` etc. to indicate which row/column it is in. A value of `0` means a cell is dead and `1` means alive.

To shift the cells right we can use `label_replace`.

label\_replace(grid, "x", "$1", "x", "^1(.\*)$")

We can do this for each of the eight neighbours, add up the values, filter if two neighbours are not alive, filter if the cell itself isn't alive and force the value to be `1`.

 (
    (grid == 1) \* 0 
    + label\_replace(grid, "x", "$1", "x", "^1(.\*)$")
    + label\_replace(grid, "x", "1$1", "x", "^(.\*)$")
    + label\_replace(grid, "y", "$1", "y", "^1(.\*)$")
    + label\_replace(grid, "y", "1$1", "y", "^(.\*)$")
    + label\_replace(label\_replace(
          grid, "y", "$1", "y", "^1(.\*)$"), "x", "$1", "x", "^1(.\*)$")
    + (label\_replace(label\_replace(
          grid, "y", "$1", "y", "^1(.\*)$"), "x", "1$1", "x", "^(.\*)$"))
    + (label\_replace(label\_replace(
          grid, "y", "1$1", "y", "^(.\*)$"), "x", "$1", "x", "^1(.\*)$"))
    + (label\_replace(label\_replace(
          grid, "y", "1$1", "y", "^(.\*)$"), "x", "1$1", "x", "^(.\*)$"))
  ) == 2
) \* 0 + 1

The `+` will automatically match time series with the exact same labels. In normal rules you'd use `+ on (x, y)` so that it's clear to the reader what labels are in play. Here it's excluded for the sake of brevity. A similar expression handles the case where 3 neighbours are alive.

## Initialization and edge cases

We need to give the grid an initial starting state. We can statically initialise init with the value `1`, and then multiply that by `0` to get an empty starting grid. 

init{x="1",y="1"} = 1
init{x="11",y="1"} = 1
init{x="1",y="11"} = 1
init{x="11",y="11"} = 1
  .
  .
  .

We can also use this to handle when an expression tries to match against a cell beyond the edge of the grid.

(label\_replace(grid, "x", "$1", "x", "^1(.\*)$") or init \* 0)

Having only dead cells at the edge and in the initial grid won't produce very exciting results. Prometheus doesn't have any good sources of randomness available, the closest thing is the `time` function so we'll use that.

(label\_replace(grid, "x", "$1", "x", "^1(.\*)$") or scalar(round((absent(nonexistent{}) \* time() + 0) % 8 / 8)))

## Visualising

Calculation is all well and good, but it'd be nice to see the grid. Console templates extend Go's templating language to let you generate custom consoles. We can use it to iterate over the grid in order and produce a HTML table.

<table>
{{ range query "grid" | sortByLabel "x" | sortByLabel "y" }}
{{ if eq .Labels.x "1" }}<tr>{{end}}
<td style="background-color: {{if eq .Value 1.0}}black{{else}}white{{end}}">&nbsp;</td>
{{ if eq .Labels.x "1111111111111111" }}</tr>{{end}}
{{ end }}
</table>

There is a [live demo](http://demo.robustperception.io:9090/consoles/life.html) available, and also the full [source code](https://github.com/RobustPerception/demo_prometheus_ansible/tree/master/roles/prometheus/files).

## Disclaimer

This is a highly esoteric example of the power of Prometheus, do not attempt anything near this complicated in production monitoring as simple monitoring is reliable monitoring. This requires features which will be in the 0.16 release of Prometheus. If you feel drowsiness, disorientation or partial blindness from reading this blog post do not look at the blog post with your remaining eye and [contact](mailto://brian.brazil@robustperception.io) your nearest Prometheus practitioner.

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
