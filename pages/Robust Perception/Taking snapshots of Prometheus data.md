---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/taking-snapshots-of-prometheus-data
author: [[Brian Brazil]] 
---
> Prometheus 2.1 added an API endpoint to take snapshots, let's see how to use it.

# Taking snapshots of Prometheus data


Prometheus 2.1 added an API endpoint to take snapshots, let's see how to use it.

As Prometheus fundamentally runs on one machine, some may wish to take backups of their data. With Prometheus 1.x this was a slow and disruptive process, requiring Prometheus to be completely restarted. The good news is that, due to its new storage engine, Prometheus 2.1 has a much better way of doing this.

To use it, you must enable the Admin API endpoints when running Prometheus:

$ ./prometheus --storage.tsdb.path=data/ --web.enable-admin-api

Then you can use a simple HTTP POST request to ask for a snapshot:

$ curl -XPOST http://localhost:9090/api/v1/admin/tsdb/snapshot
{"status":"success","data":{"name":"20180119T172548Z-78ec94e1b5003cb"}}

Here a few seconds later it has returned the name of the new snapshot in a JSON object. If you look under the `snapshots` directory of your `data` directory you'll see this snapshot:

$ cd data/snapshots
$ ls
20180119T172548Z-78ec94e1b5003cb

You can then copy this directory off to wherever you like.

The snapshots are comprised of [hard links](https://en.wikipedia.org/wiki/Hard_link) of existing blocks, and a dump of the current open blocks. As hard links are in use this means that the snapshots of older blocks take no additional disk space as there's only one copy kept on disk, however you may break Prometheus if you change them, their permissions or their user/group. When you're done you can `rm -rf` the snapshot directory, as while the snapshot takes little additional disk space initially, once the original block gets deleted/compacted the snapshot would then be what is keeping that disk space used.

To use a snapshot, make `--storage.tsdb.path` point to it.

_Want to architect your monitoring for reliability? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
