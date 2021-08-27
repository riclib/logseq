---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/left-joins-in-promql
author: [[Brian Brazil]] 
---
> Just because SQL isn't in the name doesn't mean that you can't do SQL-like joins with PromQL.

# Left joins in PromQL


Just because SQL isn't in the name doesn't mean that you can't do SQL-like joins with PromQL.

PromQL doesn't have a feature called "joins", however does have "vector matching" which is a similar idea. Aside from the syntactic differences, there's also a somewhat different set of semantics in play.

To start out let's talk about basic matching/joins in PromQL. All vector matching is done between instant vectors using binary operators, such as multiplication. So a query like

a \* b

is vector matching between two instant vectors and thus a form of join. This PromQL query can return no more samples than the smallest of the input vectors has, whereas the superficially syntactically similar SQL

SELECT a.value\*b.value, \* FROM a, b

would do a cross product. PromQL by default effectively does an inner join and requires that labels (except the metric name) on both sides of the expression to be exactly the same and match one-to-one. There's no way in PromQL to increase your cardinality (other than `absent()`, which can only go from 0 to 1). This is primarily about safety, but there's also a lack of use cases.

There's no sane way in SQL to match against all columns, you'd have to know them all in advance, so SQL has no real equivalent to `a + b`. A common use case however is only considering some labels when doing matching, so

a \* on (foo, bar) b

is broadly speaking equivalent to

SELECT a.value \* b.value, a.foo, a.bar 
FROM a INNER JOIN b ON (a.foo == b.foo AND a.bar == b.bar)

as PromQL will only return the labels which were mentioned in the `on`. This loses you all the other labels though, if you know which label you need to ignore you could have done:

a \* ignoring (baz) b

One again for the SQL equivalent you'd have to know every label/column in advance, whereas PromQL just works as it is desirable to be able to write queries that will work with any sane set of target labels.

Another difference here is that PromQL binary operators work on expressions, whereas a `SELECT` statement works on tables. So PromQL's joins are more like doing a join over two SQL subqueries rather than than two tables so can have more expressive power.

This gives us inner one-to-one joins with PromQL, but not left joins. We've also only got the matching labels in the result. We've [previously looked at](https://www.robustperception.io/how-to-have-labels-for-machine-roles) how to do some of this:

a \* on (foo, bar) group\_left(baz) b

which is equivalent to

SELECT a.value \* b.value, a.\*, b.baz
FROM a JOIN b ON (a.foo == b.foo AND a.bar == b.bar)

That is that we keep all the labels on the  left hand side, and also the `baz` label from the right hand side of the operator. This is also many-to-one matching, so there can be many samples on the left side with the same `foo` and `bar` labels, which will correspond with one value on the right.

So this is closer to a left join, but won't catch where the right hand side doesn't have a value to go with the left. This is also something we've [previously looked at](https://www.robustperception.io/existential-issues-with-metrics), and while it's a situation you want to try to avoid it is doable:

  a \* on (foo, bar) group\_left(baz) b 
or on (foo, bar)
  a

which is equivalent to

SELECT a.value \* COALESCE(b.value, 1), a.\*, b.baz
FROM a LEFT OUTER JOIN b ON (a.foo == b.foo AND a.bar == b.bar)

`or` represents the third and final type of vector matching in PromQL, many-to-many. No math is performed on values with many-to-many matching, only whether samples that match exist.

So now you know what a left join looks like in PromQL.

_Not quite able to write the PromQL query you need? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
