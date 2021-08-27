---
created: [[Aug 9th, 2021]]
type: clipping
tags: spark
source: https://medium.com/@ronbarabash/how-to-measure-consumer-lag-in-spark-structured-streaming-6c3645e45a37
author: Ron Barabash
---

# How to measure consumer lag in Spark Structured Streaming

> Ever since moving to the new Structured Streaming API we felt that open source Spark does not offer as many metrics out of the box as we…

---
Ever since moving to the new Structured Streaming API we felt that open source Spark does not offer as many metrics out of the box as we would like.

You can measure all sorts of things using the `org.apache.spark.sql.streaming.StreamingQueryProgress` class, like `processedRowsPerSecond` and `numInputRows`, but still consumer lag is one of the most important metrics to measure while running streaming jobs in high scale.

After looking online a bit we realised that we could extend the  
`StreamingQueryListener` abstract class and commit the source offsets to a **dummy consumer group** in Kafka.

From there we use a nifty little tool called [Burrow](https://github.com/linkedin/Burrow) by LinkedIn to monitor our consumers lag in InfluxDB & Grafana.

Here’s a snippet of the code stripped down and simplified:

```java
import java.util

import com.fasterxml.jackson.databind.{DeserializationFeature, ObjectMapper}
import com.fasterxml.jackson.module.scala.DefaultScalaModule
import com.yotpo.metorikku.exceptions.MetorikkuException
import org.apache.kafka.clients.consumer.{KafkaConsumer, OffsetAndMetadata}
import org.apache.kafka.common.TopicPartition
import org.apache.spark.sql.streaming.StreamingQueryListener
import org.apache.spark.sql.streaming.StreamingQueryListener._

class KafkaLagWriter(kafkaConsumer: KafkaConsumer[String, String], topic: String) extends StreamingQueryListener {
private val consumer = kafkaConsumer

def onQueryStarted(event: QueryStartedEvent): Unit = {
}

def onQueryTerminated(event: QueryTerminatedEvent): Unit = {

}

def onQueryProgress(event: QueryProgressEvent): Unit = {
	val om = new ObjectMapper().configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
	om.registerModule(DefaultScalaModule)
	event.progress.sources.foreach(source => {
 val jsonOffsets = om.readValue(source.endOffset, classOf[Map[String, Map[String, Int]]])
 jsonOffsets.keys.filter(key => key == topic)
.foreach(topic => {
  val topicPartitionMap = new util.HashMap[TopicPartition, OffsetAndMetadata]()
  val offsets = jsonOffsets.get(topic)
  offsets match {
	case Some(topicOffsetData) =>
	  topicOffsetData.keys.foreach(partition => {
		val tp = new TopicPartition(topic, partition.toInt)
		val oam = new OffsetAndMetadata(topicOffsetData(partition).toLong)
		topicPartitionMap.put(tp, oam)
	  })
	case _ =>
	  throw Exception(s"could not fetch topic offsets")
  }
  consumer.commitSync(topicPartitionMap)
})
	})
}
}
```
https://gist.github.com/RonBarabash/ef85efdfc278b37793749278f04ae59f#file-kafkalagwriter-scala

Kafka consumer is being created like so:
```java
val props = new Properties()
props.put("bootstrap.servers", "127.0.0.1:9092")
props.put("group.id", "DummyConsumerGroupID")
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
new KafkaConsumer[String, String](props)
```
https://gist.github.com/RonBarabash/f1a77f7bf9b34957e27d7e7a2bc4dfe6#file-kafkaconsumer-scala

and is being passed down once to this class in order to prevent instantiating the consumer on each callback.

This is now also an integral part of [Metorikku](https://github.com/YotpoLtd/metorikku) our open source computing engine based on Apache Spark.

you can read all about it [here](https://medium.com/yotpoengineering/introducing-metorikku-big-data-pipelines-using-apache-spark-f04456f7d5a8)