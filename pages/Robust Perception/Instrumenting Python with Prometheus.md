---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/instrumenting-python-with-prometheus
author: [[Brian Brazil]] 
---
# Instrumenting Python with Prometheus

> Python is one of the four languages that has an official Prometheus client. Let's take a quick look at how to use it.

---
Python is one of the four languages that has an official [Prometheus](https://prometheus.io/) client. Let's take a quick look at how to use it.

The first thing to do is to install the client:

```
pip install prometheus_client
```

Next you have to decide where to start instrumenting. A good way to start is to find a natural choke point such as a request router which most execution will pass through.

Let's say you had a very simple routing function:
```python
HANDLERS = {'/foo': some_function, '/bar': other_function}

def route_request(request):
  HANDLERS[request.path](request)

This is part of an online serving system, so we would like to know [request rate, errors and latency](https://prometheus.io/docs/practices/instrumentation/#online-serving-systems):

from prometheus_client import Summary, Counter

HANDLERS = {'/foo': some_function, '/bar': other_function}

REQUEST_DURATION = Summary('my_router_request_latency_seconds',
        'Latency of request router handlers', ['path'])
REQUEST_EXCEPTIONS = Counter('my_router_request_exceptions_total',
        'Exceptions thrown in request router handlers', ['path'])

def route_request(request):
    with REQUEST_DURATION.labels(request.path).time():
        with REQUEST_EXCEPTIONS.labels(request.path).count_exceptions()
            HANDLERS[request.path](request)
```
This creates a Summary to track the latency, and a Counter to track exceptions. The Summary will include the number of requests, so you don't technically have to have a separate Counter for that in order to calculate exception ratio.

This is all split out by a label on `path`, so to ensure we're exporting time series for all paths even before they're first used we should initialise them:
```python
for k in HANDLERS.keys():
   REQUEST_DURATION.labels(k)
   REQUEST_EXCEPTIONS.labels(k)

Instrumentation like this can be added throughout your codebase.

Finally we need to expose the data to Prometheus. This only needs to be done once, often in your `__main__`. The simplest way is to start up a server on a port for Prometheus, but it's also possible to expose via frameworks like Django.

from prometheus_client import start_http_server

if __name__ == '__main__':
    start_http_server(8000)
```
Prometheus is an open ecosystem, so instrumentation like the above doesn't limit you to Prometheus itself. It's just as easy to talk out to a completely different monitoring tool such as [Graphite](http://www.robustperception.io/exporting-to-graphite-with-the-prometheus-python-client/)!
