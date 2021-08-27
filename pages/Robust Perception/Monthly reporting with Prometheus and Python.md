---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/monthly-reporting-with-prometheus-and-python
author: [[Brian Brazil]] 
---
> It's common to want reports from Prometheus, such as how many requests failed over an entire month.

# Monthly reporting with Prometheus and Python


It's common to want reports from Prometheus, such as how many requests failed over an entire month.

While PromQL has some calendar functions, it's designed more for doing math over arbitrary fixed time periods rather than time periods that vary over time due to business logic. Which is to say that as different months have different numbers of days, it's not possible to do monthly reporting directly in PromQL. However this is easy with a small bit of Python scripting:

import datetime
import time
import requests  # Install this if you don't have it already.

PROMETHEUS = 'http://localhost:9090/'

# Midnight at the end of the previous month.
end\_of\_month = datetime.datetime.today().replace(day=1).date()
# Last day of the previous month.
last\_day = end\_of\_month - datetime.timedelta(days=1)
duration = '\[' + str(last\_day.day) + 'd\]'

response = requests.get(PROMETHEUS + '/api/v1/query',
  params={
    'query': 'sum by (job)(increase(process\_cpu\_seconds\_total' + duration + '))',
    'time': time.mktime(end\_of\_month.timetuple())})
results = response.json()\['data'\]\['result'\]

print('{:%B %Y}:'.format(last\_day))
for result in results:
  print(' {metric}: {value\[1\]}'.format(\*\*result))

This is intended to be run at the start of the next month local time (though all your servers run in UTC, right?), and will return the amount of CPU time each job used in the previous month. To do this we need to know when the month ended as we want to evaluate the query then, and how long the month was as that's the range to use. This will produce a result like:

December 2018:
 {'job': 'node'}: 120443.3696840266
 {'job': 'prometheus'}: 29707.01091327252
 {'job': 'pushgateway'}: 1600.0859740366413
 {'job': 'alertmanager'}: 2398.4289547078656

This is obviously a very simple example to get you going. In reality you'd probably want to work from aggregated data via recording rule, and write this information to a database of some form for prosperity.

_Want to get useful reports out of Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
