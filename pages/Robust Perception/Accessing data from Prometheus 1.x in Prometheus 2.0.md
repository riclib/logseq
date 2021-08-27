---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/accessing-data-from-prometheus-1-x-in-prometheus-2-0
author: [[Brian Brazil]] 
---
# Accessing data from Prometheus 1.x in Prometheus 2.0

> Prometheus 2.x has a different data format from 1.x, so how do you access your old data from 2.x?

---
Prometheus 2.x has a different data format from 1.x, so how do you access your old data from 2.x?

Prometheus 2.0 isn't out yet, but let's look at one of the migration issues.

Due to nature of Prometheus 1.x's storage, doing a direct migration of the data isn't very practical and the tooling for that hasn't been built. However there is a way to transparently access your old data.

The high level approach is to have the new 2.0 Prometheus transparently read data from the old 1.x Prometheus via the remote read feature.

The first step is to upgrade your 1.x Prometheus to at least version 1.8.2, so that it has the required support.

Secondly remove all of the configuration file in the 1.x Prometheus, except for `external_labels`. The old Prometheus now only exists as a data store. The entire config would look something like:
```yaml
global:
  external_labels:
    region: eu-west-1
    env: prod
```
# End of file.

Thirdly bring up a brand new Prometheus 2.0 with the full configuration file, including the same `external_labels`. (Normally every single Prometheus server should have unique `external_labels`. However as the 1.x Prometheus is no longer doing anything other than remote read and is effectively acting as remote storage, it's okay to have a duplicate). Don't forget to point the storage to a different path with `--storage.tsdb.path` and different port with `--web.listen-address`!

In addition the Prometheus 2.0 should have a `remote_read` section pointing to the 1.x Prometheus:
```yaml
global: 
  external_labels:
    region: eu-west-1
    env: prod

remote_read:
 - url: http://localhost:9090/api/v1/read
```
# Rest of config goes here.

And that's it! You can then keep the Prometheus 1.x running until the data its holding is no longer relevant. Any queries to the Prometheus 2.0 will transparently pull data from the Prometheus 1.x as needed.
