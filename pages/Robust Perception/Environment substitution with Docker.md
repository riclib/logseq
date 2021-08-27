---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/environment-substitution-with-docker
author: [[Brian Brazil]] 
---
> Prometheus leaves configuration and runtime management to their respective roles. How might you integrate those?

# Environment substitution with Docker


Prometheus leaves configuration and runtime management to their respective roles. How might you integrate those?

Some process management systems like Kubernetes and EC2 only provide certain information at runtime, often via environment variable, files in known locations, or well known URLs. Rather than trying to support everything and gradually become a configuration management system, Prometheus provides a simple standard explicit interface via flat files and SIGHUP that you can build on to meet your needs. Layering configuration management systems and templating languages rarely ends well after all. At other times while you have a fully fledged configuration management system that can construct objects as needed with little hassle, you still need a bit little of glue to run right before your application does to hook things together.

Here's an example of a Dockerfile that I threw together that does environment variable substitution before starting Prometheus for example:

FROM prom/prometheus

FROM alpine:3.10.2
RUN apk add gettext

COPY --from=0 /bin/prometheus /bin/prometheus

RUN mkdir -p /prometheus /etc/prometheus && \\
chown -R nobody:nogroup etc/prometheus /prometheus
# Run envsubst before Prometheus.
RUN echo $'#!/bin/sh\\n\\
envsubst < /etc/prometheus/orig.yml > /etc/prometheus/prometheus.yml && \\
exec /bin/prometheus "$@"' \\
> /etc/prometheus/entrypoint.sh
RUN chmod +x /etc/prometheus/entrypoint.sh
ENTRYPOINT \["/etc/prometheus/entrypoint.sh"\]

CMD \[ "--config.file=/etc/prometheus/prometheus.yml", \\
"--storage.tsdb.path=/prometheus" \]
USER nobody
EXPOSE 9090
VOLUME \[ "/prometheus" \]
WORKDIR /prometheus

# This is your local prometheus.yml.
ADD prometheus.yml /etc/prometheus/orig.yml

`envsubst` is part of gettext, and a startup script applies this to the config before starting Prometheus in the usual way. To use it have a `prometheus.yml` locally, such as the very minimal:

scrape\_configs:
- job\_name: "$HOSTNAME"

And then you can build and run with:

docker build -t envprom . && docker run --rm -p9090:9090 envprom

If you then visit [http://localhost:9090/config](http://localhost:9090/config) you'll see something like

[![](https://www.robustperception.io/wp-content/uploads/2019/09/Screenshot_2019-09-24_15-12-05.png)](https://www.robustperception.io/wp-content/uploads/2019/09/Screenshot_2019-09-24_15-12-05.png)  
where `$HOSTNAME` has been substituted.

This is only one way to do it. You could have the config file stored elsewhere, you might use `sed` to preform more complex operations, or you could pull data from elsewhere. If you're on Kubernetes you might use init containers rather than combining everything into one container, or adjust the command line that the Kubelet ultimately executes. There's no reason that the config file you have in source control must match exactly what lands on the machine running the application, indeed in a sophisticated setup the ultimate file may be completely generated - such as in the Prometheus Operator.

As a final note while it has gained some popularity over the past several years, [putting secrets in environment variables is unwise](https://movingfast.io/articles/environment-variables-considered-harmful/). This is one of those things you want to handle at deploy or run time using approaches such as the above.

_Wondering how to best manage Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
