---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/checking-openmetrics-output-is-valid
author: [[Brian Brazil]] 
---
> The Python client can be used to check if a given metrics output is valid OpenMetrics format.

# Checking OpenMetrics output is valid


The Python client can be used to check if a given metrics output is valid OpenMetrics format.

The OpenMetrics format is near completion with the last few details still being determined, but if you're working on an implementation you can check if what you have is valid as the draft spec currently stands. The Prometheus Python client contains the reference implementation of the OpenMetrics text format, and it's parser can be used to check that a given output is valid.

Install the latest version of the Python client if you don't already have it:

pip3 install --upgrade prometheus\_client

Then you can pipe your data into the following one-liner:

python3 -c 'import sys; from prometheus\_client.openmetrics import parser; list(parser.text\_string\_to\_metric\_families(sys.stdin.buffer.read().decode("utf-8")))'

So for example doing

echo -e "a 1" | ...

Will produce the error `Missing # EOF at end` and

echo -e "a 1\\n# EOF" | ...

would return nothing and have an exit code of 0.

As the format isn't fully finalised yet that your output is valid today doesn't automatically mean it'll be okay next month, but it will point out more obvious problems.

_Wondering how best to expose your metrics? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
