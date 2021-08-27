---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus
source: https://www.robustperception.io/atomic-writes-and-the-textfile-collector
author: [[Brian Brazil]]
---

> To avoid weirdness, write your files atomically.

# Atomic Writes and the Textfile Collector

To avoid weirdness, write your files atomically.

[Previous](https://www.robustperception.io/using-the-textfile-collector-from-a-shell-script) [posts](https://www.robustperception.io/monitoring-directory-sizes-with-the-textfile-collector) [involving](https://www.robustperception.io/quick-sensor-metrics-with-the-textfile-collector) the node exporter's textfile collector have not written directly to the `.prom` file, rather writing first to a temporary file and then renaming it. This may seem like an unnecessary complication as things may seem fine without it, however it's actually an important detail for correctness.

Let's say you're using `>` file redirection in shell to write to your `.prom` file. The `>` means to overwrite the given file with the contents of standard output (as against `>>` which will append). To do so first your shell must open the file, which it will do passing the `O_CREAT` and `O_TRUNC` flags to `open()`. `O_CREAT` will create the file if it doesn't already exist, and `O_TRUNC` will truncate the size of the file to zero so one way or the other we will end up with a zero-sized file. Once the file is opened successfully, the content of standard output can be appended using as many `write()` calls as are needed.

While POSIX says that other processes should never see partial results of a `write()` call, if the node exporter was to receive a scrape and read the `.prom` file at any point between the file being opened and the last write being completed then it would see partial content. This is undesirable, as it could result in no data, partial data, or an error: any of which will likely cause weirdness downstream.

How can we avoid this? The usual way is to take advantage of `rename()` allowing the changing of a file name, and in doing so replacing of one file with another in a way that doesn't allow partial files to be visible. So you can write out the new content to a temporary file, and then rename it to the final filename.

There is a caveat in that `rename()` only works for files on the same filesystem, so if you use `mv` across filesystems (such as if your `/tmp` is on a different filesystem) then this may not work as desired. This is why it's usual to have the temporary file in the same directory as the file you are trying to update.

All of this is presuming standard POSIX semantics, which tend to be less than certain when non-local filesystems such as NFS are involved. How different network filesystems and vendor appliances deal with uses like these can vary by a surprising amount, which is the main reason that NFS is the only filesystem Prometheus explicitly does not support. Not that the textfile collector is ever likely to be interacting with a non-local filesystem.

_Have questions on monitoring cronjobs? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
