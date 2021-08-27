---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/temperature-and-hardware-monitoring-metrics-from-the-node-exporter
author: [[Brian Brazil]] 
---
> The node exporter exposes the various hardware monitoring metrics of Linux, including temperature, fans, and voltages.

# Temperature and hardware monitoring metrics from the node exporter


The node exporter exposes the various hardware monitoring metrics of Linux, including temperature, fans, and voltages.

You've probably come across the `sensors` command from lm-sensors which on my desktop produces output like:

coretemp-isa-0000
Adapter: ISA adapter
Package id 0: +46.0°C (high = +85.0°C, crit = +105.0°C)
Core 0: +39.0°C (high = +85.0°C, crit = +105.0°C)
Core 1: +42.0°C (high = +85.0°C, crit = +105.0°C)
Core 2: +47.0°C (high = +85.0°C, crit = +105.0°C)
Core 3: +44.0°C (high = +85.0°C, crit = +105.0°C)

acpitz-virtual-0
Adapter: Virtual device
temp1: +27.8°C (crit = +106.0°C)
temp2: +29.8°C (crit = +106.0°C)

asus-isa-0000
Adapter: ISA adapter
cpu\_fan: 0 RPM

The node exporter has the same information under `node_hwmon_*`. This machine only happens to have temperature and a (non-reporting) fan, so let's look at those first.

Temperature is the main metric and is in `node_hwmon_temp_celsius`:

\# HELP node\_hwmon\_temp\_celsius Hardware monitor for temperature (input)
# TYPE node\_hwmon\_temp\_celsius gauge
node\_hwmon\_temp\_celsius{chip="acpitz",sensor="temp1"} 27.8
node\_hwmon\_temp\_celsius{chip="acpitz",sensor="temp2"} 29.8
node\_hwmon\_temp\_celsius{chip="platform\_coretemp\_0",sensor="temp1"} 47
node\_hwmon\_temp\_celsius{chip="platform\_coretemp\_0",sensor="temp2"} 40
node\_hwmon\_temp\_celsius{chip="platform\_coretemp\_0",sensor="temp3"} 42
node\_hwmon\_temp\_celsius{chip="platform\_coretemp\_0",sensor="temp4"} 47
node\_hwmon\_temp\_celsius{chip="platform\_coretemp\_0",sensor="temp5"} 42

There's also `node_hwmon_temp_crit_celsius` and `node_hwmon_temp_crit_alarm_celsius` for alerting, however in a Prometheus setup you'd usually have thresholds in your alerting rules.

Those names aren't quite what `sensors` is showing. While lm-sensors does some tweaking of names in the background, the same raw information is in another two metrics from the node exporter:

\# HELP node\_hwmon\_sensor\_label Label for given chip and sensor
# TYPE node\_hwmon\_sensor\_label gauge
node\_hwmon\_sensor\_label{chip="platform\_coretemp\_0",label="core\_0",sensor="temp2"} 1
node\_hwmon\_sensor\_label{chip="platform\_coretemp\_0",label="core\_1",sensor="temp3"} 1
node\_hwmon\_sensor\_label{chip="platform\_coretemp\_0",label="core\_2",sensor="temp4"} 1
node\_hwmon\_sensor\_label{chip="platform\_coretemp\_0",label="core\_3",sensor="temp5"} 1
node\_hwmon\_sensor\_label{chip="platform\_coretemp\_0",label="package\_id\_0",sensor="temp1"} 1
node\_hwmon\_sensor\_label{chip="platform\_eeepc\_wmi",label="cpu\_fan",sensor="fan1"} 1# HELP node\_hwmon\_chip\_names Annotation metric for human-readable chip names
# TYPE node\_hwmon\_chip\_names gauge
node\_hwmon\_chip\_names{chip="acpitz",chip\_name="acpitz"} 1
node\_hwmon\_chip\_names{chip="platform\_coretemp\_0",chip\_name="coretemp"} 1
node\_hwmon\_chip\_names{chip="platform\_eeepc\_wmi",chip\_name="asus"} 1

Using `group_left` you can join the `label` and `chip_name` labels in PromQL, though beware that they may be missing in some cases so you need to be resilient to that.

For fans there's `node_hwmon_fan_rpm` for the speed, and `node_hwmon_pwm_enable` to indicate if pulse-width modulation (a way of controlling fan speed) is enabled.

Voltage will be under `node_hwmon_in_volts` and `node_hwmon_cpu_volts`. Current is `node_hwmon_curr_amps`.

Power usage can be reported directly by the hardware, in which case it'll be `node_hwmon_energy_watt`. It can also an energy counter `node_hwmon_energy_joule_total`, and you'll get power in Watts (which are joules per second) if you use PromQL's `rate()` on it.

The humidity ratio can be found in `node_hwmon_humidity`.

`node_hwmon_fault` and `node_hwmon_alarm` can indicate hardware issues, and `node_hwmon_beep_enabled` if beeping is enabled.

As these metrics are coming from a wide range of different hardware and associated software, the metrics available will vary across machines. For example some metrics may have a `input` in the name, or have additional metrics indicating the minimum and maximum values over some time period. Support for more metrics is likely to be added to the node exporter over time too.

_Want to know if something other than LP0 is on fire? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
