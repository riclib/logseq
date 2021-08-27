---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/dude-wheres-my-exporter
author: [[Brian Brazil]] 
---
> So you have just discovered Prometheus and want to try it out or use it to replace your old monitoring system but have run into a part of your stack that you cannot instrument with a client library and for which there are no officially supported exporters. What do you do?

# Dude, where’s my exporter?


So you have just discovered Prometheus and want to try it out or use it to replace your old monitoring system but have run into a part of your stack that you cannot instrument with a client library and for which there are no officially supported exporters. What do you do?

Thankfully, you're in luck as the Prometheus team curates a list of both official and non-official exporters in the docs that you can view over on the [Prometheus documentation](https://prometheus.io/docs/instrumenting/exporters/). How do these exporters compare to their official counterparts? Unfortunately the team cannot vet each and every exporter for all of the best practices we follow for the official exporters, however we do try to only list the best exporter for any given piece of tech if there are multiple exporters written for it.

But what do you do if there is no exporter listed for the tech you wish to monitor? You could search through GitHub and see if it exists as the list may not yet contain the desired exporter or you could go ahead and write your own.

But how does one go about doing that? The Prometheus team has written up [a guide](https://prometheus.io/docs/instrumenting/writing_exporters/) on how to write Prometheus exporters as well as best practices. As most exporters are open source, you can also fork a similar exporter to the one you wish to create and work from there.

If you're writing a new exporter, please try to have the default port unique. We maintain a tracked list of default port numbers for Prometheus exporters and other components [here](https://github.com/prometheus/prometheus/wiki/Default-port-allocations). This also serves as a fairly comprehensive list of all the exporters that are out there.

_Need the expert's advice on monitoring your systems? [Contact us](mailto:prometheus@robustperception.io)._

Published by Conor Broderick in [Posts](https://www.robustperception.io/category/posts)
