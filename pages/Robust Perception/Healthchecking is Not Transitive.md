---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/healthchecking-is-not-transitive
author: [[Brian Brazil]] 
---
# Healthchecking is Not Transitive

> A blog on monitoring, scale and operational Sanity

---
A blog on monitoring, scale and operational Sanity

Systems such as Consul perform [healthchecking](https://www.consul.io/intro/getting-started/checks.html) of local services and expose this information to other machines within the cluster. Does this mean that the service will work when you try to talk to it?

With systems such as Consul it can be tempting to think that your job is done reliability-wise due to its in-built health checking abilities. Having just one daemon checking health saves potential clients from having to duplicate that work, but just because a service works when checked from the local machine doesn't mean it'll work from other machines.

[![Healthchecking is Not Transitive](http://www.robustperception.io/wp-content/uploads/2015/09/Healthchecking-is-Not-Transitive.png)](http://www.robustperception.io/wp-content/uploads/2015/09/Healthchecking-is-Not-Transitive.png)

There may be a firewall in the way, or the server mightn't bound to all the right interfaces or there could even be a partial network partition due to misconfigured routers.

You don't want to be sending your requests off to fail, so what can you do?

-   In advance of the client needing to do an RPC, the client should open a connection and do a healthcheck
-   Regularly perform communication over this connection (such as healthchecks) to detect any faults that develop
-   If the connection or heathchecks fail, don't use that server until they work again
-   Perform requests as usual over the connection
