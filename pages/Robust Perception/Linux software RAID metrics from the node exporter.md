---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/linux-software-raid-metrics-from-the-node-exporter
author: [[Brian Brazil]] 
---
> /proc/mdstat is another of the files that the node exporter exposes as metrics.

# Linux software RAID metrics from the node exporter


`/proc/mdstat` is another of the files that the node exporter exposes as metrics.

The Linux software RAID metrics are one of the more intricate metrics in terms of parsing due to `/proc/mdstat` being more suited to humans than machines, which you can get a sense of from the [unittest fixture](https://github.com/prometheus/procfs/blob/fce2797c0a7014c95e72ad044cf057299f801660/fixtures.ttar#L1968-L2023). For one of my home RAID1 arrays the node exporter produces:

\# HELP node\_md\_blocks Total number of blocks on device.
# TYPE node\_md\_blocks gauge
node\_md\_blocks{device="md0"} 3.3538048e+07
# HELP node\_md\_blocks\_synced Number of blocks synced on device.
# TYPE node\_md\_blocks\_synced gauge
node\_md\_blocks\_synced{device="md0"} 3.3538048e+07
# HELP node\_md\_disks Number of active/failed/spare disks of device.
# TYPE node\_md\_disks gauge
node\_md\_disks{device="md0",state="active"} 2
node\_md\_disks{device="md0",state="failed"} 0
node\_md\_disks{device="md0",state="spare"} 0
# HELP node\_md\_disks\_required Total number of disks of device.
# TYPE node\_md\_disks\_required gauge
node\_md\_disks\_required{device="md0"} 2
# HELP node\_md\_state Indicates the state of md-device.
# TYPE node\_md\_state gauge
node\_md\_state{device="md0",state="active"} 1
node\_md\_state{device="md0",state="inactive"} 0
node\_md\_state{device="md0",state="recovering"} 0
node\_md\_state{device="md0",state="resync"} 0

`node_md_disks` is the primary metric of interest as you will want to know if there are any failed disks, and depending on your the setup you may also know that there must be a minimum number of spares and/or active devices. A RAID1 array with only one active disk is not exactly in prime health after all.

`node_md_state` indicates if the array is recovering using a spare, resyncing after an event such as an unclean shutdown, active and healthy, or disabled and inactive. When you are recovering/resyncing, `node_md_blocks` and `node_md_blocks_synced` can tell you how the array getting synced back up is progressing (and if `/proc/sys/dev/raid/speed_limit_max` may need a tweak). There are not something to alert on, but could be useful for graphing rather than having to eyeball `mdadm --detail`, and also to get some of the history of the array.

_Need help monitoring hardware? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
