---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/dealing-with-too-many-open-files
author: [[Brian Brazil]] 
---
> While not a problem specific to Prometheus, being affected by the open files ulimit is something you're likely to run into at some point.

# Dealing with “too many open files”


While not a problem specific to Prometheus, being affected by the open files ulimit is something you're likely to run into at some point.

Ulimits are an old Unix feature that allow limiting how much resources a user uses, such as processes, CPU time, and various types of memory. You can view your shell's current ulimits with `ulimit -a`:

$ ulimit -a
core file size (blocks, -c) 0
data seg size (kbytes, -d) unlimited
scheduling priority (-e) 0
file size (blocks, -f) unlimited
pending signals (-i) 63796
max locked memory (kbytes, -l) unlimited
max memory size (kbytes, -m) unlimited
**open files (-n) 1024**
pipe size (512 bytes, -p) 8
POSIX message queues (bytes, -q) 819200
real-time priority (-r) 95
stack size (kbytes, -s) 8192
cpu time (seconds, -t) unlimited
max user processes (-u) 63796
virtual memory (kbytes, -v) unlimited
file locks (-x) unlimited

The one of interest here is open files, which is 1024 on my desktop. To find the limit for applications instrumented with Prometheus there's metrics for the limit and current usage:

\# HELP process\_max\_fds Maximum number of open file descriptors.
# TYPE process\_max\_fds gauge
process\_max\_fds 1024
# HELP process\_open\_fds Number of open file descriptors.
# TYPE process\_open\_fds gauge
process\_open\_fds 65

And Prometheus itself also logs the limits at startup (it's the soft limit that matters):

level=info ts=... caller=main.go:225 fd\_limits="(soft=1024, hard=1048576)"

For other currently running processes you can use `prlimit` or look in `/proc/PID/limits`.

A limit of 1024 per process is something I'm unlikely to run into in day to day desktop usage, but a server like Prometheus which may have sockets open to hundreds of targets, HTTP connections to service discovery, graph requests coming in, and data files open could between them hit this. When a process runs out of file descriptors, it tends not to ends well and Prometheus is not unusual in this regard. It and other Go programs will get a "too many open files" error when this happens.

So how do you increase this limit to avoid this? The first thing to be aware of is that increasing a ulimit beyond the hard limit will require elevated privileges, and thus changing ulimits permanently usually involves changing files in `/etc`.

On an Ubuntu system the limits usually come from `/etc/security/limits.conf` where you can set the `nofile` limit across users or groups. If you're using systemd then `/etc/systemd/system.conf` has a `DefaultLimitNOFILE` you can specify across all units, or you can set it on a per-unit basis with `LimitNOFILE` in the `Service` section. In an emergency, you can also use `prlimit` to change the limits for a running process.

Something to watch out for is that you can end up with different ulimits if you restart a process from a SSH session versus having it started at boot. Accordingly it's wise to always double check what your ulimits actually are to ensure your desired configuration has been applied.

As to what to set the file ulimit to, I'd suggest something large like a million that you're unlikely to ever hit. File descriptors are not a scarce resource.

_Want to improve Prometheus reliability? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
