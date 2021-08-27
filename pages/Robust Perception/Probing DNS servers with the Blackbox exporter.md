---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/probing-dns-servers-with-the-blackbox-exporter
author: [[Brian Brazil]] 
---
> Among the Blackbox exporter's probe types is DNS.

# Probing DNS servers with the Blackbox exporter


Among the Blackbox exporter's probe types is DNS.

As with other probe types, DNS probes are intended to be used to run one query against many different DNS servers by Prometheus, and verify that the DNS servers are working. Let's try it out by checking if DNS servers are returning that robustperception.io is using Google for receiving email:

wget https://github.com/prometheus/blackbox\_exporter/releases/download/v0.12.0/blackbox\_exporter-0.12.0.linux-amd64.tar.gz
tar -xzf blackbox\_exporter-\*.linux-amd64.tar.gz
cd blackbox\_exporter-\*
cat > blackbox.yml << EOF
modules:
  dns\_rp\_mx:
    prober: dns
    dns:
      query\_name: "robustperception.io"
      query\_type: "MX"
      validate\_answer\_rrs:
        fail\_if\_not\_matches\_regexp:
         - "robustperception.io.\\t.\*\\tIN\\tMX\\t.\*google.\*"
EOF
./blackbox\_exporter

If you visit [http://localhost:9115/probe?module=dns\_rp\_mx&target=8.8.8.8](http://localhost:9115/probe?module=dns_rp_mx&target=8.8.8.8) this will be tested against Google's Public DNS, and should succeed. In reality you'd usually point this at your own authoritative DNS servers using a sample query rather than verifying that a public DNS service is working - though you could also use it like this to check that records like MX are present. For A and AAAA records you're generally better off using an icmp/tcp/http probe to see if the service as a whole is working, which will implicitly also test DNS. As mentioned in a [previous](https://www.robustperception.io/debugging-blackbox-exporter-failures) post, you can also add the debug parameter to see the full DNS response: [http://localhost:9115/probe?module=dns\_rp\_mx&target=8.8.8.8&debug=true](http://localhost:9115/probe?module=dns_rp_mx&target=8.8.8.8&debug=true).

You can then use this in your `prometheus.yml` as you would any other Blackbox module:

scrape\_configs:
  - job\_name: 'blackbox\_dns'
    metrics\_path: /probe
    params:
      module: \[dns\_rp\_mx\]
    static\_configs:
      - targets:
        - 8.8.4.4  # Test various public DNS providers are working.
        - 8.8.8.8
        - 1.0.0.1
        - 1.1.1.1
    relabel\_configs:
      - source\_labels: \[\_\_address\_\_\]
        target\_label: \_\_param\_target
      - source\_labels: \[\_\_param\_target\]
        target\_label: instance
      - target\_label: \_\_address\_\_
        replacement: 127.0.0.1:9115

[Other features](https://github.com/prometheus/blackbox_exporter/blob/master/CONFIGURATION.md#dns_probe) of the DNS prober include being able to select TCP or UDP, checking the response code, and checking authoritative and additional resource records that are returned.

_Have questions about the Blackbox exporter? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
