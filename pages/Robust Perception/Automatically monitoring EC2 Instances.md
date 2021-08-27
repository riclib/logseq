---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/automatically-monitoring-ec2-instances
author: [[Brian Brazil]] 
---
# Automatically monitoring EC2 Instances

> Having to manually update a list of machines in a configuration file gets annoying after a while. One of the features of Prometheus is service discovery, allowing you to automatically discover and monitor your EC2 instances!

---
Having to manually update a list of machines in a configuration file gets annoying after a while. One of the features of Prometheus is service discovery, allowing you to automatically discover and monitor your EC2 instances!

To start with, I spun up two EC2 instances running the [node exporter](http://www.robustperception.io/machine-monitoring-with-prometheus-debs/). I ensured that ports 9100 and 9090 were open in the Security Group.

[![](http://www.robustperception.io/wp-content/uploads/2015/12/Screenshot-011215-203641.png)](http://www.robustperception.io/wp-content/uploads/2015/12/Screenshot-011215-203641.png)

Two instances running the node exporter

Next I setup an IAM user with the `AmazonEC2ReadOnlyAccess` policy, noting down the access and secret key. This policy gives the user permission to list EC2 instances.

[![](http://www.robustperception.io/wp-content/uploads/2015/12/Screenshot-011215-203514.png)](http://www.robustperception.io/wp-content/uploads/2015/12/Screenshot-011215-203514.png)

IAM user with AmazonEC2ReadOnlyAccess policy

Finally I installed Prometheus on one of the machines, with the following configuration in `/etc/prometheus/prometheus.yml`:
```yaml

global:
  scrape_interval: 1s
  evaluation_interval: 1s

scrape_configs:
  - job_name: 'node'
    ec2_sd_configs:
      - region: eu-west-1
        access_key: PUT_THE_ACCESS_KEY_HERE
        secret_key: PUT_THE_SECRET_KEY_HERE
        port: 9100
```
Visiting `/consoles/index.html` on port 9090 of the Prometheus server I see my two nodes have been automatically discovered:

[![](http://www.robustperception.io/wp-content/uploads/2015/12/Screenshot-011215-204313.png)](http://www.robustperception.io/wp-content/uploads/2015/12/Screenshot-011215-204313.png)

This is what we wanted, but it'd be nice if we could have the instance ID displayed rather than the private IP address. Maybe we also want to only scrape some nodes based on a tag.

Relabelling gives you the power to customise what labels Prometheus applies to your target. For an example, I put the following into `/etc/prometheus/prometheus.yml` and reloaded Prometheus:
```yaml

global:
  scrape_interval: 1s
  evaluation_interval: 1s

scrape_configs:
  - job_name: 'node'
    ec2_sd_configs:
      - region: eu-west-1
        access_key: PUT_THE_ACCESS_KEY_HERE
        secret_key: PUT_THE_SECRET_KEY_HERE
        port: 9100
    relabel_configs:
        # Only monitor instances with a Name starting with "SD Demo"
      - source_labels: [__meta_ec2_tag_Name]
        regex: SD Demo.*
        action: keep
        # Use the instance ID as the instance label
      - source_labels: [__meta_ec2_instance_id]
        target_label: instance
```yaml

Visiting the console again, I see that it has worked perfectly! The same technique can be used to monitor instances in an auto-scaling group.

[![](http://www.robustperception.io/wp-content/uploads/2015/12/Screenshot-011215-205009.png)](http://www.robustperception.io/wp-content/uploads/2015/12/Screenshot-011215-205009.png)

As always with Prometheus you can drill down to get more details information about each instance:

[![](http://www.robustperception.io/wp-content/uploads/2015/12/Screenshot-011215-205317.png)](http://www.robustperception.io/wp-content/uploads/2015/12/Screenshot-011215-205317.png)

Check out the [documentation](http://prometheus.io/docs/operating/configuration/) if you'd like to learn more about relabelling and service discovery.
