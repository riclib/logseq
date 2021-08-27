---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/using-the-java-client-with-gradle
author: [[Brian Brazil]] 
---
> While the Java client library uses pom.xml and Maven, there's nothing stopping you from using other tools such as Gradle

# Using the Java client with Gradle


While the Java client library uses pom.xml and Maven, there's nothing stopping you from using other tools such as Gradle

The [example dependencies for the Java client](https://github.com/prometheus/client_java#assets) are for Maven, if you have an existing Gradle project the equivalents will look something like:

dependencies {
 compile group: 'io.prometheus', name: 'simpleclient', version: '0.5.0'
 compile group: 'io.prometheus', name: 'simpleclient\_hotspot', version: '0.5.0'
 compile group: 'io.prometheus', name: 'simpleclient\_httpserver', version: '0.5.0'
}

If you're starting from scratch, I've created a [sample project](https://github.com/RobustPerception/prometheus_examples/tree/master/java_examples/java_gradle) to get you going. After ensuring Gradle is installed:

git clone https://github.com/RobustPerception/prometheus\_examples
cd prometheus\_examples/java\_examples/java\_gradle/
gradle jar
java -jar ./build/libs/java\_gradle.jar

If you now visit [http://localhost:8000/metrics](http://localhost:8000/metrics) you'll see your metrics!

_Want to learn more about instrumenting Java? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
