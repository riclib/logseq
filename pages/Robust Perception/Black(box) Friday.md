---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/blackbox-friday
author: [[Conor Broderick]] 
---
# Black(box) Friday

> While eager consumers flock to retailers both online and in-store for big savings during the infamous day after Thanksgiving known as Black Friday, one must wonder the cost incurred by companies accommodating both the crowds swarming their store floors and the increase in traffic from online shoppers.

---
While eager consumers flock to retailers both online and in-store for big savings during the infamous day after Thanksgiving known as Black Friday, one must wonder the cost incurred by companies accommodating both the crowds swarming their store floors and the increase in traffic from online shoppers.

In this blogpost we used [Prometheus](https://prometheus.io/) and the [Blackbox exporter](https://github.com/prometheus/blackbox_exporter) to observe the increase in latency experienced by some of the top online retailers in the US and UK.

In order to measure the latencies, I spun up an EC2 instance in us-east-1 (North Virginia) and another in eu-west-2 (London). Each instance was running the Blackbox exporter configured to probe the top shopping sites of each region respectively as well as a Prometheus to ingest the created metrics.

Looking at the UK first, we have the latencies measured for Amazon.co.uk over a 48 hour period starting at midnight on Black Friday and ending midnight on Sunday:

[![](https://www.robustperception.io/wp-content/uploads/2017/11/Screen-Shot-2017-11-27-at-16.02.24.png)](https://www.robustperception.io/wp-content/uploads/2017/11/Screen-Shot-2017-11-27-at-16.02.24.png)

As we can see from the above graph, there was a significant increase in latency to Amazon.co.uk on Black Friday that did not return to normal levels until about 2pm that day. This would indicate that the site saw a significant increase in load on Black Friday, perhaps due to the large number of shoppers visiting the site to avail of the best deals before they sold out similar to the stampedes one might see in a mall on the same day.

Now let's take a look the breakdown of this metric using the metric probe_http_duration_seconds which is a [httptrace](https://golang.org/pkg/net/http/httptrace/) based probe which allows us to gather fine-grained information throughout the lifecycle of an HTTP client request.

Black Friday - Saturday:

[![](https://www.robustperception.io/wp-content/uploads/2017/11/Screen-Shot-2017-11-27-at-16.07.13.png)](https://www.robustperception.io/wp-content/uploads/2017/11/Screen-Shot-2017-11-27-at-16.07.13.png)

In this case we can see that the "processing" and "tls" portions of the HTTP probe have contributed the most to the increase in latency and may be worth looking at to reduce this latency for future spikes in traffic.

How did things fair stateside? (Note: times have been adjusted to Eastern Time Zone (ETZ) (UTC−5:00))

Black Friday - Saturday:

[![](https://www.robustperception.io/wp-content/uploads/2017/11/Screen-Shot-2017-11-27-at-16.08.12.png)](https://www.robustperception.io/wp-content/uploads/2017/11/Screen-Shot-2017-11-27-at-16.08.12.png)

Again, we notice a significant increase in latency during Black Friday in the morning, at midday, and again in the evening when shoppers would be hitting target.com to check on the deals.
