---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/negative-lookahead-assertions-in-promql-selectors
author: [[Brian Brazil]] 
---
> RE2 doesn't support lookahead, but that doesn't mean you can't do it in another way.

# Negative lookahead assertions in PromQL selectors


[RE2](https://github.com/google/re2/wiki/Syntax) doesn't support lookahead, but that doesn't mean you can't do it in another way.

Negative lookahead assertions are a feature of Perl Compatible Regular Expressions that says don't match if the following thing matches, without consuming any of the text. This is commonly used to provide an exception to a regex.  For example `(?!bar)b..` would match `baz` but not `bar`. For [performance reasons](https://github.com/google/re2/issues/156) RE2, which Go and thus Prometheus use, don't support this and many other Perl regex extensions.

One way around simpler cases like this is to spell out the exception more explicitly using character classes, for example `b([^a].|a[^r])`. This works, but gets a little tedious and can be hard to read.

However in PromQL selectors, regexes aren't used once in isolation. A given label name can have multiple matchers referring to it, and both regex matcher and negative regex matchers are available. So this use case could be handled as `metric{foo=~"b..",foo!~"bar"}`, which is also likely more readable to non-regex experts.

That a given use case isn't directly possible doesn't mean that there isn't another way to do it.

_Have PromQL questions? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
