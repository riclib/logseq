---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/pre-creating-alertmanager-silences
author: [[Brian Brazil]] 
---
> You don't have to wait for alerts to fire to create a silence.

# Pre-creating Alertmanager Silences


You don't have to wait for alerts to fire to create a silence.

While silences are useful when an alert is firing and you want to keep your pager quiet while investigating or waiting for your intervention to take effect, that's not the only way they can be used. If you've an upcoming maintenance, or automated procedure such as taking a backup that would cause alerts to fire, you can use silences in advance to avoid pager spam.

The easiest way to create a silence in advance is by going to the Alertmanager UI, clicking on "New Silence", entering in the appropriate details and clicking "Create". That's fine for ad-hoc silence creation, but not really viable when automation is involved.

Similarly to how Prometheus has promtool, the Alertmanager has amtool. If you wanted to create a 4 hour silence for backups on a machine in a shell script you could use:

SILENCE\_ID=$(amtool --alertmanager.url http://alertmanager:9093/ silence add --duration=4h --comment "Backups on $HOSTNAME" job=myjob instance=$HOSTNAME:1234)

Which will take the id of the created silence from stdout and put it in a variable. Once the backup is done you can expire the silence:

amtool --alertmanager.url http://alertmanager:9093/ silence expire ${SILENCE\_ID}

As you are manually removing the silence once you are done, it's safe to have a long conservative duration for the silence so that if the backup takes a little longer than usual you don't end up getting paged. If however the backup job gets killed in such a way that the silence is not expired early, it'll still expire by itself after a while.

_Want to reduce spammy notifications? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
