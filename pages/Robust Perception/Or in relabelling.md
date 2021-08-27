---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/or-in-relabelling
author: [[Brian Brazil]] 
---
> How do you allow for the keep relabel action halting relabelling for things not kept?

# Or in relabelling


How do you allow for the `keep` relabel action halting relabelling for things not kept?

Any targets dropped by the `drop` relabel action won't be processed by any following relabel actions. The same applies for `keep`, if it's not kept then it's now gone. In practice this means that a list of actions like

\- source\_labels: \[label\]
  regexp: foo
  action: keep
- source\_labels: \[label\]
  regexp: bar
  action: keep

will return nothing, as `label` cannot be both `foo` and `bar` at the same time.

You could combine the actions using the regex alternation operator:

\- source\_labels: \[label\]
  regexp: foo|bar
  action: keep

This can however get a little unwieldy when you want to apply more intricate business logic across multiple source labels.

Another approach would be to set a flag when your criteria is met, and use `keep` on that:

\- source\_labels: \[label\]
  regexp: foo
  target\_label: \_\_tmp\_keep\_me
  replacement: true
- source\_labels: \[label\]
  regexp: bar
  target\_label: \_\_tmp\_keep\_me
  replacement: true
- source\_labels: \[\_\_tmp\_keep\_me\]
  regex: true
  action: keep

The default `replace` action only has an effect if it matches, so if `label` is `foo` or `bar` the `__tmp_keep_me` label ends up with the value `true` which a `keep` action can then be predicated on. This pattern can be used with any number of actions.

The `__tmp` prefix is reserved for user usage in target relabelling, and it is promised that no future Prometheus feature will use this prefix. As is begins with `__` it will be discarded with all other metadata before determining the final target labels.

_Have questions about relabelling? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
