---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/creating-alertmanager-silences-from-python
author: [[Brian Brazil]] 
---
> We recently looked at creating silences from the command line, what about from programs?

# Creating Alertmanager Silences from Python


We [recently looked](https://www.robustperception.io/pre-creating-alertmanager-silences) at creating silences from the command line, what about from programs?

While shell scripting for automated tasks is common, beyond a certain point their complexity justifies a different programming language be employed - such as Python. While you could shell out to amtool to create silences, there's also a simple [REST API](http://petstore.swagger.io/?url=https://raw.githubusercontent.com/prometheus/alertmanager/master/api/v2/openapi.yaml#/silence/postSilences) you can use. Python's date libraries makes this a little trickier than it could be, but the equivalent of the previous post's example is:

#!/usr/bin/python3
import requests
import socket
import datetime
import time

res = requests.post("http://alertmanager:9093/api/v2/silences", json={
    "matchers": \[
        {"name": "job", "value": "myjob", "isRegex": False},
        {"name": "instance", "value": "{}:1234".format(socket.gethostname()), "isRegex": False},
        \],
    "startsAt": datetime.datetime.utcfromtimestamp(time.time()).isoformat(),
    "endsAt": datetime.datetime.utcfromtimestamp(time.time() + 4\*3600).isoformat(),
    "comment": "Backups on {}".format(socket.gethostname()),
    "createdBy": "My backup script",
    },
    )
res.raise\_for\_status()
silenceId = res.json()\["silenceID"\]

Once the script has finished its maintenance, you can then use the silence's id to remove the silence:

res = requests.delete("http://alertmanager:9093/api/v2/silence/{}".format(silence))
res.raise\_for\_status()

This Python code is doing the same as thing as amtool, so choose whichever approach makes the most sense for your situation.

_Wondering how to monitor batch jobs? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
