---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/numbers-from-displaystrings-with-the-snmp_exporter
author: [[Brian Brazil]] 
---
> Not all numbers in SNMP are exposed as numbers, which is why regex_extracts exists.

# Numbers from DisplayStrings with the snmp_exporter


Not all numbers in SNMP are exposed as numbers, which is why `regex_extracts` exists.

As a metrics-based monitoring system Prometheus works off numbers, with labels to help group related things. SNMP's data model is surprisingly close to the Prometheus data model, with SNMP table indexes basically being Prometheus instrumentation labels. So in theory once you have a MIB you could automatically convert from SNMP. Unfortunately many devices and MIBs follow the letter of the SNMP spec, however do not follow the spirit thus requiring special handling for some MIBs and devices. This is okay if you are writing code for one particular device, but not so much if you're trying to support SNMP in general.

SNMP doesn't have a floating point type (there is a draft specification which a few vendors and the [SNMP exporter support](https://github.com/prometheus/snmp_exporter/pull/268), but it's not common), only integers. This is fine for bytes or packets, but less so for things like temperature where you may wish to indicate that a device is 27.8°C.

There's two common approaches to this. One is to use an SNMP gauge and expose deci-Celcius, for example the number 278 to indicate 27.8°C. This is a little annoying to work with as it's not a base unit, but it is workable as PromQL can do basic arithmetic.

The other is to expose a DisplayString (an ASCII string of up to 255 characters) with the value like `27.8`, which is more problematic. The general principle behind the SNMP exporter and generator is that you provide the objects you'd like exposed, and the rest will Just Work. If the syntax of an SNMP object is a DisplayString then the SNMP exporter will do the best thing it can do with that, and that is to expose it as a label value. When the string is actually a numeric metric value this is useless to PromQL as labels are opaque strings, and may even cause cardinality issues as the value changes over time.

This is a common enough occurrence that there's support in the SNMP exporter for handling it via `regex_extracts`. This allows doing a regular expression substitution on the string (not too different from relabelling), and then trying to parse the result as a floating point number.

The simplest case is where the string is a float, with no additional clutter:

overrides: 
  exampleObject:
    regex\_extracts: 
      '': 
        - regex: '(.\*)' 
          value: '$1'

This goes in the `overrides` section of your generator.yml as do other things were the MIB doesn't contain sufficient/correct information. `exampleObject` is the object name, the metric name suffix is the empty string, all the string value is matched, and then parsed as a floating point value. If the strings looked like `27.8C` you could use a regex of `(.*)C` instead.

The use of a map with an empty key, and then a list of regexes replacements may seem a bit over-engineered at first glance. This is unfortunately not so. A given string may actually contain more than one number, rather than following the spirit of SNMP and having an object each. So you can have multiple metrics extracted, with the key of the map added as a suffix on the object name so that the resulting samples don't clash. For example CPU and system temperatures might be reported together as `27.8C 28.4C`:

overrides: 
  exampleObject:
    regex\_extracts: 
      'CPU': 
        - regex: '(.\*) .\*' 
          value: '$1'
      'System': 
         - regex: '.\* (.\*)' 
           value: '$1'

So here you'd end up with metrics called `exampleObjectCPU` and `exampleObjectSystem`.

There can also be cases where the string is indeed a string, but as it changes over time would be more useful as a number. For example `on` and `off` could be more useful as 1 and 0:

overrides: 
  exampleObject:
    regex\_extracts: 
      '': 
         - regex: 'on' 
           value: '1'
         - regex: 'off' 
           value: '0'

These are the simple and more common cases, regular expressions should allow for more complicated bespoke formats to also be handled.

_Have questions about SNMP and Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
