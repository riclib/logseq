---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/instrumenting-java-with-prometheus
author: [[Brian Brazil]] 
---
# Instrumenting Java with Prometheus

> Getting started with a new library it helps to have an example to work from. Let's look at a simple example of using Prometheus instrumentation in Java.

---
Getting started with a new library it helps to have an example to work from. Let's look at a simple example of using Prometheus instrumentation in Java.

I've written a small [example webserver](https://github.com/RobustPerception/java_examples/tree/master/java_simple). Let's download it and run it:

git clone https://github.com/RobustPerception/java_examples.git
cd java_examples/java_simple/
mvn package
java -jar target/java_simple-1.0-SNAPSHOT-jar-with-dependencies.jar

If you visit [http://localhost:1234/](http://localhost:1234/) you'll see Hello World, and on [http://localhost:1234/metrics](http://localhost:1234/metrics) you can see the metrics.

Notice how `hello_worlds_total` increments every time you get a Hello World. This is an example of a Counter, one of the basic Prometheus metric types. So how does it work?

[![](http://www.robustperception.io/wp-content/uploads/2016/01/Screenshot-050116-190636.png)](http://www.robustperception.io/wp-content/uploads/2016/01/Screenshot-050116-190636.png)

One of the metrics on the /metrics

## Counter

There's two statements that are important in [`ExampleServlet`](https://github.com/RobustPerception/java_examples/blob/master/java_simple/src/main/java/io/robustperception/java_examples/JavaSimple.java#L17-L29):
```java
 static final Counter requests = Counter.build()
   .name("hello_worlds_total")
   .help("Number of hello worlds served.").register();
```
This creates the metric which is shared across all instances of the object. It gives it a name and includes some help text, so that those using the metric later on will know what it means.
```java
requests.inc();
```
This simple statement increments the counter by one. So with just two statements you can count anything you like!

## Setup

There's a small bit of setup work that need to be done once, no matter how many metrics you have. To expose the metrics used in your code, we [add the Prometheus servlet to our Jetty server](https://github.com/RobustPerception/java_examples/blob/master/java_simple/src/main/java/io/robustperception/java_examples/JavaSimple.java#L39):

context.addServlet(new ServletHolder(new MetricsServlet()), "/metrics");

You may have noticed that there were many other useful metrics included about the JVM and process. These come from several classes, but it's [only one line to use them](https://github.com/RobustPerception/java_examples/blob/master/java_simple/src/main/java/io/robustperception/java_examples/JavaSimple.java#L41):
```java
 DefaultExports.initialize();
```
For these to work, you'll need to add [a few dependencies](https://github.com/RobustPerception/java_examples/blob/master/java_simple/pom.xml#L11-L25) to your `pom.xml`:
```xml
<dependency>
  <groupId>io.prometheus</groupId>
  <artifactId>simpleclient</artifactId>
  <version>0.0.11</version>
</dependency>
<dependency>
  <groupId>io.prometheus</groupId>
  <artifactId>simpleclient_hotspot</artifactId>
  <version>0.0.11</version>
</dependency>
<dependency>
  <groupId>io.prometheus</groupId>
  <artifactId>simpleclient_servlet</artifactId>
  <version>0.0.11</version>
</dependency>
```
Now that you know the basics, try adding Prometheus instrumentation to your own code!
