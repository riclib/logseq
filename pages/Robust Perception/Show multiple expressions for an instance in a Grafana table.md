---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/show-multiple-expressions-for-an-instance-in-a-grafana-table
author: [[Brian Brazil]] 
---
> Have you ever wanted to have a table showing multiple metrics across all of your instances?

# Show multiple expressions for an instance in a Grafana table


Have you ever wanted to have a table showing multiple metrics across all of your instances?

I'm going to show you how to show CPU usage and RSS for all of your instances, here I'm using Grafana 7.1.1 and the end result will look something like:

[![](https://www.robustperception.io/wp-content/uploads/2020/10/Screenshot_2020-10-15_12-43-56.png)](https://www.robustperception.io/wp-content/uploads/2020/10/Screenshot_2020-10-15_12-43-56.png)

First create a Table panel, and define your queries. The key settings are that the Format should be `Table`, and the queries are `Instant`:

[![](https://www.robustperception.io/wp-content/uploads/2020/10/Screenshot_2020-10-15_12-38-27.png)](https://www.robustperception.io/wp-content/uploads/2020/10/Screenshot_2020-10-15_12-38-27.png)

Next in Transform do an `Outer join` on the `instance` label, and then `Organize fields` to hide and rename columns:

[![](https://www.robustperception.io/wp-content/uploads/2020/10/Screenshot_2020-10-15_12-39-37.png)](https://www.robustperception.io/wp-content/uploads/2020/10/Screenshot_2020-10-15_12-39-37.png)

Finally to add some polish, set units and limit decimals via the Overrides in the Options pane:

[![](https://www.robustperception.io/wp-content/uploads/2020/10/Screenshot_2020-10-15_12-41-43.png)](https://www.robustperception.io/wp-content/uploads/2020/10/Screenshot_2020-10-15_12-41-43.png)

This can be expanded to add additional columns and metrics.

_Looking to have more useful dashboards? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
