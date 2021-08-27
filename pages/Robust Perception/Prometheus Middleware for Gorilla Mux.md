---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/prometheus-middleware-for-gorilla-mux
author: [[Brian Brazil]] 
---
> Your HTTP router is usually the best place to measure your application latency.

# Prometheus Middleware for Gorilla Mux


Your HTTP router is usually the best place to measure your application latency.

If you ever find yourself wanting to use the same metric from multiple files, that's usually a sign that either they should in fact be different metrics or that the metric belongs higher in the call stack. For example rather than having each HTTP handler have the responsibility to measure its latency, with the code duplication and muddy semantics that implies, do the instrumentation up in your HTTP router instead.

[Gorilla's Mux](https://www.gorillatoolkit.org/pkg/mux) is a HTTP router you might be using if your application is in Go, and it offers the possibility to inject middleware to allow you to do just this:

package main
  
import (
  "net/http"

  "github.com/gorilla/mux"
  "github.com/prometheus/client\_golang/prometheus"
  "github.com/prometheus/client\_golang/prometheus/promauto"
  "github.com/prometheus/client\_golang/prometheus/promhttp"
)

var (
  httpDuration = promauto.NewHistogramVec(prometheus.HistogramOpts{
    Name: "myapp\_http\_duration\_seconds",
    Help: "Duration of HTTP requests.",
  }, \[\]string{"path"})
)

// prometheusMiddleware implements mux.MiddlewareFunc.
func prometheusMiddleware(next http.Handler) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r \*http.Request) {
    route := mux.CurrentRoute(r)
    path, \_ := route.GetPathTemplate()
    timer := prometheus.NewTimer(httpDuration.WithLabelValues(path))
    next.ServeHTTP(w, r)
    timer.ObserveDuration()
  })
}

func main() {
  r := mux.NewRouter()
  r.Use(prometheusMiddleware)
  r.Path("/metrics").Handler(promhttp.Handler())
  r.Path("/obj/{id}").HandlerFunc(
    func(w http.ResponseWriter, r \*http.Request) {})

  srv := &http.Server{Addr: "localhost:1234", Handler: r}
  srv.ListenAndServe()
}

This middleware uses the path template, so the label value will be `/obj/{id}` rather than `/obj/123` which would risk a cardinality explosion. Depending on your use case you could expand this to include other aspects of the request such as the host, method, and URL parameters - or add additional metrics like bytes transferred.

In this trivial example the Prometheus middleware is the only middleware. If you have multiple layers of middleware, then instrumentation should be first. Imagine if you instead had an auth layer before the instrumentation layer, you'd not capture the latency added by the auth layer and even worse could miss rejected requests completely.

_Unsure how to best instrument? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
