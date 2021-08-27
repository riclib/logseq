---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus, monitoring, time series
source: https://www.robustperception.io/exposing-the-software-version-to-prometheus
author: [[Brian Brazil]]
---
>  You shouldn't have the version of your software as either a target label, or exposed via a label on all metrics of your server as it'll make using the metrics more challenging. Use _info labels instead

# Exposing the Software Version to Prometheus

I've previously mentioned[^1] that you shouldn't have the version of your software as either a target label, or exposed via a label on all metrics of your server as it'll make using the metrics more challenging. What should you do instead?

The way to approach this is the similar to machine roles[^2]. ==You expose a single time series with all the labels you want, and the value `1`==. For example Prometheus itself exports a time series called `prometheus_build_info`:

  ```
  prometheus\_build\_info{branch="HEAD",goversion="go1.6.2",
      revision="16d70a8b6bd90f0ff793813405e84d5724a9ba65",version="1.0.1"} 1
  ```
To export a metric like this with Python we'd do:

  ```
  build\_info = Gauge('prometheus\_build\_info', 'Build information', 
      \['branch', 'goversion', 'revision', 'version'\])
  build\_info.labels('HEAD', 'go1.6.2', 
      '16d70a8b6bd90f0ff793813405e84d5724a9ba65', '1.0.1').set(1)
  ```

This isn't much use on its own, how can we slice and aggregate based on it?
==Selecting all instances with a particular label can be done with the `and` operator==, which will return the left hand side if the labels on the right hand side has matching labels. For example to display the number of time series for all Prometheus servers running 1.0.1:

  ```
  prometheus\_local\_storage\_memory\_series{job="prometheus"}
  and on (instance, job) prometheus\_build\_info{job="prometheus",version="1.0.1"}
  ```
  
If you want ==to add the `version` label to a time series you can do==:

  ```prometheus\_local\_storage\_memory\_series{job="prometheus"}
  * on (instance, job) group\_left(version)
    prometheus\_build\_info{job="prometheus"}
  ```
To top it off, let's find the average number of time series per Prometheus version:

  ```
  avg by (version)(
      prometheus\_local\_storage\_memory\_series{job="prometheus"}
    \* on (instance, job) group\_left(version)
      prometheus\_build\_info{job="prometheus"}
  )
  ```

This is just a small taste of what's possible. This technique can be extended to information beyond software version!

- [^1] [[Target labels are for life, not just for Christmas]]
- [^2] [[How to have labels for machine roles]]