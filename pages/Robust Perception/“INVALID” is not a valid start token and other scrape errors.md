---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/invalid-is-not-a-valid-start-token-and-other-scrape-errors
author: [[Brian Brazil]] 
---
> What can you do to pin down metrics-parsing related scrape errors in Prometheus?

# “INVALID” is not a valid start token and other scrape errors


What can you do to pin down metrics-parsing related scrape errors in Prometheus?

Not all scrape errors are due to network or HTTP issues. There's also a variety of parsing errors that are possible including `"INVALID" is not a valid start token`, `strconv.ParseFloat: parsing "...": invalid syntax`, `expected timestamp or new record, got "MNAME"` and `expected label name, got "INVALID"`. There's a number of common causes of these such as having hyphens or periods in metric names, numbers at the start of label or metric names, and missing the newline on the last line of the output.

While spotting these by hand is possible, there's an easier way to figure out which line is the issue. `promtool` which comes with Prometheus has a subcommand to check metrics syntax. Let's download it and try it out:

wget https://github.com/prometheus/prometheus/releases/download/v2.7.1/prometheus-2.7.1.linux-amd64.tar.gz
tar -xzf prometheus-\*.tar.gz
cd prometheus-\*
echo 'a.b 1' | ./promtool check metrics

Which will produce output like:

error while linting: text format parsing error in line 1: expected float as value, got ".b"

indicating that the period in the metric name is the issue.

If the trailing newline was missing it'd look like:

$ echo -n 'a 1' | ./promtool check metrics
error while linting: text format parsing error in line 1: unexpected end of input stream

Once you know what the problem is, you can fix the bug in the relevant server so that it produces valid Prometheus text output.

Beyond checking if metrics output can be parsed, this also acts as a linter. For example:

echo 'a 1' | ./promtool check metrics
a no help text

Usually you don't use echo, but fetch the metrics via HTTP:

$ GET http://demo.robustperception.io:9100/metrics | ./promtool check metrics

The lack of output here indicates that all is well, and the exit code will also be 0.

_Need help exposing metrics? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
