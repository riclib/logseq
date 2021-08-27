---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/using-external-urls-and-proxies-with-prometheus
author: [[Brian Brazil]] 
---
> Sometimes users will not access Prometheus's UI directly, instead using another URL. How do you make this work?

# Using external URLs and proxies with Prometheus


Sometimes users will not access Prometheus's UI directly, instead using another URL. How do you make this work?

The Prometheus web UI, such as the expression browser, depends on it being accessed on the same URL on which Prometheus itself is listening. This is as Prometheus needs to know where assets such as Javascript are and what URL to use in any links or redirects. If Prometheus is behind a reverse proxy, particularly one where Prometheus is not at the root, this doesn't work so well.

The good news is that Prometheus has features to cover this exact use case, and the Alertmanager also has the exact same features. This is primarily configured by the `--web.external-url` flag.

Let's say you have Prometheus running on it's usual port, and Nginx installed with the following configuration:

http {
 server {
   listen 0.0.0.0:19090;
   location /prometheus/ {
     proxy\_pass http://localhost:9090/prometheus/;
   }
 }
}
events {
}

This configuration would serve Prometheus up on [http://localhost:19090/prometheus/](http://localhost:19090/prometheus/), and Nginx will include the `/prometheus/` prefix when passing on requests to Prometheus. To make this work you'd need to run Prometheus like:

prometheus --web.external-url http://localhost:19090/prometheus/

You should be aware that with this external URL, the `/prometheus/` path prefix will be required for all HTTP access to Prometheus. The /metrics will be on [http://localhost:9090/prometheus/metrics](http://localhost:9090/prometheus/metrics) for example.

In reality you wouldn't just be serving up Prometheus up on a different port on the local machine as in this demonstration. It'll likely be via a more human-readable DNS name, and possibly with HTTPS and other features in use. The principles all remain the same though!

_Have questions around Prometheus and proxies? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
