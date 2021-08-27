---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/external-urls-and-path-prefixes
author: [[Brian Brazil]] 
---
> In a previous post I looked at setting the external URL. What if the reverse proxy is sending a different path than the user is using?

# External URLs and path prefixes


In a [previous post](https://www.robustperception.io/using-external-urls-and-proxies-with-prometheus/) I looked at setting the external URL. What if the reverse proxy is sending a different path than the user is using?

By default Prometheus (and Alertmanager) presumes that any path in the external URL is a prefix path that'll be in all requests that are sent to it. However that's not always the case, and the `--web.route-prefix` flag allows you to control this more granularly.

Let's say you have Prometheus running on it's usual port, and Nginx installed with the following configuration:

http {
 server {
   listen 0.0.0.0:19090;
   location /prometheus/ {
     proxy\_pass http://localhost:9090/;
   }
 }
}
events {
}

Unlike the example in the [previous post](https://www.robustperception.io/using-external-urls-and-proxies-with-prometheus/), this will strip off the `/prometheus/` before passing on requests to Prometheus. In this case you need to specify the URL the user is using in their browser, and also that the prefix Prometheus will see in it's HTTP requests is not `/prometheus/` but rather just the empty `/`:

prometheus --web.external-url http://localhost:19090/prometheus/ --web.route-prefix=/

As before, Prometheus will be accessible on [http://localhost:19090/prometheus/](http://localhost:19090/prometheus/).

_Have questions around Prometheus and proxies? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
