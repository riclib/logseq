---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/avoid-irate-in-alerts
author: [[Brian Brazil]] 
---
# Avoid irate() in alerts

> While the irate() function is useful for granular graphs, it is not suitable for alerting.

---
While the `irate()` function is useful for granular graphs, it is not suitable for alerting.

`irate()` takes in a counter and calculates the per-second increase based on the two most recent samples in the range. By contrast, `rate()` looks at all the samples in the range.

The outcome of this is that with `irate()` you can see all the dips and spikes with the same resolution as that of the scrape. This is also where `irate()` falls down for alerting.

Say that you have a alert with an expression of `irate(my_counter[1m]) > 10`  and a `for: 5m`, which is to say that the per-second rate is over 10 for 5 minutes. If the rate increases to 15 per second you'd expect this to fire in 5 minutes or so.

However things aren't so simple. Rarely is it the case that are metrics perfectly steady, especially when things are in an abnormal state. Sure the average might be 15 per second, but it might be 9.5 one instant and 18.7 a moment later. That 9.5 will reset the `for: 5m` alert so now you have to start counting again. Removing the `for: 5m` wouldn't help, as it'd lead to an overly sensitive alert that'd have lots of false positives.

`rate()` avoids this as it is an average over many samples, so it is resilient to brief dips and spikes.

Accordingly always use `rate()` in alerts, not `irate()`.

_Want advice on alert thresholds? [Contact us](mailto:prometheus@robustperception.io)._
