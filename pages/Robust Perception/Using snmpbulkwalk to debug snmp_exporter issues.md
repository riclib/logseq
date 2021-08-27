---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/using-snmpbulkwalk-to-debug-snmp_exporter-issues
author: [[Brian Brazil]] 
---
> Many problems with the snmp_exporter turn out to actually be issues elsewhere, but how can you tell?

# Using snmpbulkwalk to debug snmp_exporter issues


Many problems with the snmp\_exporter turn out to actually be issues elsewhere, but how can you tell?

SNMP can be bit mysterious at times, and sometimes the device just doesn't respond or gives weird results. You might see a scrape error like:

An error has occurred during metrics gathering:

error collecting metric Desc{fqName: "snmp\_error", help: "Error scraping target", constLabels: {}, variableLabels: \[\]}: error getting target 192.168.1.2: Request timeout (after 3 retries)

A good first step is determining if the issue is how the snmp\_exporter is being used, or if the issue is with communicating to the device in general. One of the things that can cause this is that you have the wrong authentication settings. To check this you can run:

snmpbulkwalk -v2c -c public -Cr25 -On 192.168.1.2 1.3.6.1.2.1.2.2

On failure you'll likely see something like:

Timeout: No Response from 192.168.1.2

indicating usually an authentication or networking issue, and thus a problem that's not within the snmp\_exporter.

Let's look at the flags we're using:

`-v2c` says to use SNMP v2, there's also v1 (in which case use the slower `snmpwalk` instead) and v3 (which has more complicated auth options, [the docs](https://github.com/prometheus/snmp_exporter/blob/a622a7902eabf5e4a76b96ebbfd9b9ff789662b0/generator/README.md#file-format) explain how to map your generator config to snmp command line options).

`-c public` says the community string (password) is the typical default of public. If this is incorrect, then you'll get no response and it'll look just as if the device doesn't exist or is firewalled off.

`-Cr25` says to request batches of up to 25 objects, which is the snmp\_exporter default. Unfortunately many devices do not implement this correctly, so try smaller numbers if you're getting weird or failed results. The `-On` requests numeric output, which makes weird results such as missing objects easier to spot. If you find this is a problem you can adjust `max_repetitions`, but a lower number will slow down scrapes. Conversely a larger number will speed things up, and on a correctly implemented SNMP agent is harmless if the device doesn't support the larger number.

If everything is working you'll see output like:

.1.3.6.1.2.1.2.2.1.1.1 = INTEGER: 1
.1.3.6.1.2.1.2.2.1.1.2 = INTEGER: 2
.1.3.6.1.2.1.2.2.1.1.3 = INTEGER: 3
.1.3.6.1.2.1.2.2.1.1.4 = INTEGER: 4
.1.3.6.1.2.1.2.2.1.1.5 = INTEGER: 5
.1.3.6.1.2.1.2.2.1.1.6 = INTEGER: 6
.1.3.6.1.2.1.2.2.1.1.7 = INTEGER: 7

and so on. If not, try adjusting the above until things are working and then copy the updated settings over to your configuration files.

Here I'm using `1.3.6.1.2.1.2.2` which is `ifTable`, where all the standard network interface stats are and which every SNMP agent should have. If you are having issues with a different oid, take the oid you need from the `walk` part of your configuration file.

You may notice that some oids are listed under `get` rather than `walk`. While it's far less likely that a device implements `GET` incorrectly (as against `GETBULK`, which `snmpbulkwalk` uses), you can test these with `snmpget`:

 snmpget -v2c -c public -On 192.168.1.2 1.3.6.1.2.1.1.3.0

This is the same as the above, but without the `-Cr` flag. You may get a `No Such Instance currently exists at this OID` or `No Such Object available on this agent at this OID` error message. This is not a problem per-se, and means you have successfully talked to the device but nothing was found.

_Have questions about network monitoring? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
