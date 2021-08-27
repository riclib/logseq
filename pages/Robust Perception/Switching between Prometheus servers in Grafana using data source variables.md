---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/switching-between-prometheus-servers-in-grafana-using-data-source-variables
author: [[Brian Brazil]] 
---
> Having to maintain dashboards for every Prometheus server you have would be a bit annoying. Thankfully Grafana has a feature for this.

# Switching between Prometheus servers in Grafana using data source variables


Having to maintain dashboards for every Prometheus server you have would be a bit annoying. Thankfully Grafana has a feature for this.

Variables in Grafana (previously known as templates) allow parameterisation of a dashboard via a drop-down menu. Often this is used to switch between machines or services, so that you can have per-machine dashboards without needing to create a dashboard every time a new machine appears. They're also stored in URL parameters, so could be linked from alert notifications or wiki pages.

One type of variable is the data source variable, which as the name indicates lets data sources be parameterised. I have setup some Prometheus data sources for demonstration in a Grafana 6.2.5:

[![](https://www.robustperception.io/wp-content/uploads/2019/07/Screenshot_2019-07-01_13-31-57.png)](https://www.robustperception.io/wp-content/uploads/2019/07/Screenshot_2019-07-01_13-31-57.png)

Then I create a new dashboard, and go to its settings:

[![](https://www.robustperception.io/wp-content/uploads/2019/07/Screenshot_2019-07-01_13-34-53.png)](https://www.robustperception.io/wp-content/uploads/2019/07/Screenshot_2019-07-01_13-34-53.png)

I Add variable, with the name Source, the type of the variable is Datasource and the type of data source is Prometheus:

[![](https://www.robustperception.io/wp-content/uploads/2019/07/Screenshot_2019-07-01_13-37-15.png)](https://www.robustperception.io/wp-content/uploads/2019/07/Screenshot_2019-07-01_13-37-15.png)

All the Prometheus data sources are listed, plus the "default" data source. Here I only want the prom- Prometheus servers, so I can put in an Instance name filter regular expression of `prom-.*`:

[![](https://www.robustperception.io/wp-content/uploads/2019/07/Screenshot_2019-07-01_13-40-22.png)](https://www.robustperception.io/wp-content/uploads/2019/07/Screenshot_2019-07-01_13-40-22.png)

By having a standard naming scheme for data sources, it's possible to select just the ones I want. For example I might use the pattern prom-ENV-SERVICE-LOCATION, likely following the information contained in `external_labels`.

I Add the variable, go back to the dashboard, and add a simple query of `up` to the initial panel, and set the Query to be against $Source:

[![](https://www.robustperception.io/wp-content/uploads/2019/07/Screenshot_2019-07-01_13-45-18.png)](https://www.robustperception.io/wp-content/uploads/2019/07/Screenshot_2019-07-01_13-45-18.png)

If I now go back to the dashboard the Source is there as a dropdown, and changing it will switch which data source is queried:

[![](https://www.robustperception.io/wp-content/uploads/2019/07/Screenshot_2019-07-01_13-46-01.png)](https://www.robustperception.io/wp-content/uploads/2019/07/Screenshot_2019-07-01_13-46-01.png)

There's also `&var-Source=prom-dev-ie` in my browser's URL, which I could also change if I wanted to link to the dashboard with a specific data source.

By using this approach it's easy to have a central Grafana that pulls data from all your individual Prometheus servers, with dashboards that can look at whichever is the required Prometheus with a click. This is a simple but powerful setup.

_Wondering how to best organise your dashboards? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
