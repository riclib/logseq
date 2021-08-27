---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/prometheus-security-authentication-authorization-and-encryption
author: [[Brian Brazil]] 
---
# Prometheus Security: Authentication, Authorization and Encryption

> It's a frequently asked question as to how to do various security related features with Prometheus. Let's take a deeper look at why we chose the approach we did.

---
It's a [frequently asked question](https://prometheus.io/docs/introduction/faq/#why-don't-the-prometheus-server-components-support-tls-or-authentication?-can-i-add-those?) as to how to do various security related features with Prometheus. Let's take a deeper look at why we chose the approach we did.

The Prometheus project takes the stance that server side security features are outside its scope. This is one of our most questioned product decisions. To see why we made this choice, let's consider a world where we did handle security in Prometheus and accepted some of the PRs we regularly receive to add it.

Starting off someone sends a PR to add [basic authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) to Prometheus. It's just a username and password coming from flags, so that's only a few lines of code.

That's got a few security problems though. Firstly we're putting a password on the command line, and secondly we're sending a password in the clear over the network.  We'll need to at least store a [hashed and salted version of the password](https://www.owasp.org/index.php/Password_Storage_Cheat_Sheet), and keep up to date as hash best practices changed.

We also need to add encryption of data over the wire, which means [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security). There are at present [18 different configuration options](https://golang.org/pkg/crypto/tls/#Config) for TLS in Go. Of these there are 13 that an end user could plausibly want to configure today, and potentially more in the future as the protocol evolves and new security threats emerge.

So that's basic auth covered. Then there's also [Digest access](https://en.wikipedia.org/wiki/Digest_access_authentication), [bearer tokens](https://en.wikipedia.org/wiki/OAuth), [Kerberos](https://en.wikipedia.org/wiki/Kerberos_(protocol)), [GSSAPI](https://en.wikipedia.org/wiki/Generic_Security_Services_Application_Program_Interface), [OpenID](https://en.wikipedia.org/wiki/OpenID_Connect), [SSL client certificates](https://en.wikipedia.org/wiki/Transport_Layer_Security#Client-authenticated_TLS_handshake), [Radius](https://en.wikipedia.org/wiki/RADIUS) and [PAM](https://en.wikipedia.org/wiki/Pluggable_authentication_module) - each with their own nuances and evolution over time.

Then there's authorization. Authentication tells you who someone is, but we also need to know if they're allowed to access a given endpoint. Maybe there's different basic auth users for different levels of access, or talk to [OpenLDAP](http://www.openldap.org/), or [ActiveDirectory](https://en.wikipedia.org/wiki/Active_Directory), or Kerberos, or MySQL, or PostgresQL - the configuration and settings of each are likely to be organisation specific.

Going a step deeper, should we prohibit certain types of query? Challenging given PromQL is [Turing Complete](http://www.robustperception.io/conways-life-in-prometheus/). Maybe time series based access control then?

Security is a fractally complicated space, and on top of that it is ever changing. Prometheus as a project could decide to take this support into our core and provide a coherent and secure system - however that would be a substantial and Sisyphean task.

Prometheus is a monitoring system, not a security framework. Accordingly given the massive work that'd be involved in creating and maintaining a security framework, we've decided to instead leave the task up to 3rd party systems such as [Nginx](http://www.robustperception.io/adding-basic-auth-to-prometheus-with-nginx/) and Apache. These already implement the security feature that our users ask for, are well maintained, and allow the Prometheus project to focus on its core goal of monitoring.

