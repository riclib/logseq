---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/converting-string-states-to-booleans-with-the-jmx-exporter
author: [[Brian Brazil]] 
---
> You may have come across a mBean attribute with a value such as STARTED or RUNNING. How do you convert that to a more useful boolean metric?

# Converting string states to booleans with the JMX exporter


You may have come across a mBean attribute with a value such as STARTED or RUNNING. How do you convert that to a more useful boolean metric?

In [Exposing version numbers with the JMX exporter](https://www.robustperception.io/exposing-version-numbers-with-the-jmx-exporter/) you saw how you could expose a version number that was the string value of an mBean attribute as a label on a metric. This works fine for version numbers which don't change over the lifetime of the JVM, but not so well with states that change as you want to avoid [existential issues with metrics](https://www.robustperception.io/existential-issues-with-metrics/). You can however extend the approach to convert these strings into a boolean gauge indicating whether the state is currently RUNNING.

Continuing on the example let's say that you wanted a jvm\_openjdk metric which was 1 if you were using a OpenJDK JVM and 0 otherwise. This can be done with:

\---
rules:
 - pattern: 'java.lang<type=Runtime, key=java.runtime.name><>SystemProperties: .\*OpenJDK.\*'
   name: jvm\_openjdk
   value: 1
 - pattern: 'java.lang<type=Runtime, key=java.runtime.name><>SystemProperties: .\*'
   name: jvm\_openjdk
   value: 0
 

This will produce metric output like:

\# HELP jvm\_openjdk java.util.Map<java.lang.String, java.lang.String> (java.lang<type=Runtime, key=java.runtime.name><>SystemProperties)
# TYPE jvm\_openjdk untyped
jvm\_openjdk 1.0

How this works is that each mBean attribute is tested against the pattern in each rule in turn, and the first one that matches wins. Thus if the string contains OpenJDK it'll match the first rule and have a value of 1, otherwise the second rule which matches any value for the attribute will apply and it'll have a value of 0. This does meant that you can produce at most one metric from a given mBean attribute.

This technique can be used with any other state strings exposed via JMX!

_Want to learn more about how to monitor JVM applications? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
