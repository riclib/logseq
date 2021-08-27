---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/using-the-textfile-collector-from-a-shell-script
author: [[Brian Brazil]] 
---
> The node exporter's textfile collector is handy for monitoring machine-level cronjobs. How would you go about that?

# Using the textfile collector from a shell script


The node exporter's textfile collector is handy for monitoring machine-level cronjobs. How would you go about that?

Let's say you had a simple bash script and you wanted to track how long it takes, and when it last ran. You could do something like:

#!/bin/bash

# Adjust as needed.
TEXTFILE\_COLLECTOR\_DIR=/var/lib/node\_exporter/textfile\_collector/
# Note the start time of the script.
START="$(date +%s)"

# Your code goes here.
sleep 10

# Write out metrics to a temporary file.
END="$(date +%s)"
cat << EOF > "$TEXTFILE\_COLLECTOR\_DIR/myscript.prom.$$"
myscript\_duration\_seconds $(($END - $START))
myscript\_last\_run\_seconds $END
EOF

# Rename the temporary file atomically.
# This avoids the node exporter seeing half a file.
mv "$TEXTFILE\_COLLECTOR\_DIR/myscript.prom.$$" \\
  "$TEXTFILE\_COLLECTOR\_DIR/myscript.prom"

Once you ensure that the `--collector.textfile.directory` flag on the node exporter is set to the right directory you're good to go. You could add any more metrics you think are useful, such as whether the script succeeded.

The node exporter exposes a metric called `node_textfile_mtime_seconds` which indicates when each textfile collector file was last modified, which can be useful for detecting if a cronjob hasn't run in a while.

Instead of using a temporary file, you could also use sponge from [moreutils](https://joeyh.name/code/moreutils/) to handle that for you.

_Wondering how best to monitor batch jobs? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
