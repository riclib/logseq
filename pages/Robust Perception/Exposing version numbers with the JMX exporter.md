---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/exposing-version-numbers-with-the-jmx-exporter
author: [[Brian Brazil]] 
---
> While the JMX attributes you are usually working with will be numbers, the JMX exporter also has support for strings.

# Exposing version numbers with the JMX exporter


While the JMX attributes you are usually working with will be numbers, the JMX exporter also has support for strings.

The JMX exporter automatically handles mBean attributes with numeric and boolean values. You will however sometimes come across string attributes that are of interest, such as version numbers. While these aren't exposed out of the box, it is easy to configure the JMX exporter to handle these.

There are two aspects of the JMX exporter's design that are relevant. The first is that the regex `pattern` is applied across both the name of the mBean attribute and its value, and the second is that you can override the value of a metric. If you wanted to expose the JVM's runtime name, which is in the `java.lang<type=Runtime, key=java.runtime.name><>SystemProperties` attribute, you could do:

\---
rules:
 - pattern: 'java.lang<type=Runtime, key=java.runtime.name><>SystemProperties: (.\*)'
   name: jvm\_runtime
   value: 1
   labels:
     runtime: "$1"

The value here is after the colon and space, and the value is placed in the `runtime` label. This metric will have a value of 1, and on /metrics will look like:

\# HELP jvm\_runtime java.util.Map<java.lang.String, java.lang.String> (java.lang<type=Runtime, key=java.runtime.name><>SystemProperties)
# TYPE jvm\_runtime untyped
jvm\_runtime{runtime="OpenJDK Runtime Environment",} 1.0

In reality there is not much use for this particular rule, as the `jvm_info` metric which is automatically exposed by the JMX exporter already includes the runtime. However you can apply the same approach to your application's own mBeans.

_Want to better instrument Java applications? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
