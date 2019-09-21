# 인기 검색어 순위 만들기 (Kafka Spark Dstreaming)-(2)

###Spark dstreaming 예제 실습



이전에 spark를 통해 elasticsearch에 데이터를 삽입한 것 처럼 다시 환경을 구축하고 배포까지 테스트를 해보겠다.



spark 홈페이지 api

https://spark.apache.org/docs/latest/streaming-programming-guide.html



###Dependency 추가 하기 

Grade

```groovy
implementation 'org.apache.spark:spark-streaming_2.11:2.4.3'
```

maven

```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming_2.12</artifactId>
    <version>2.4.3</version>
    <scope>provided</scope>
</dependency>
```

Kafka, flume, kenesis 등은 spark-streaming core에 포함되어있지않아 , 별도로 의존성을 추가해줘야한다.

```txt
For ingesting data from sources like Kafka, Flume, and Kinesis that are not present in the Spark Streaming core API, you will have to add the corresponding artifact spark-streaming-xyz_2.12 to the dependencies. For example, some of the common ones are as follows.
```

| Source  | Artifact                                                     |
| ------- | ------------------------------------------------------------ |
| Kafka   | spark-streaming-kafka-0-10_2.12</br>https://spark.apache.org/docs/2.2.0/streaming-kafka-0-8-integration.html |
| Flume   | spark-streaming-flume_2.12                                   |
| Kinesis | spark-streaming-kinesis-asl_2.12 [Amazon Software License]   |



###SPARK CODE



kafka에서 데이터를 받아와야하기 때문에 아래와 같은 의존성라이브러리를 또 추가 

예전에는 kafka 0.8 버전을 사용하였는데 kafka 0.10 버전으로 사용하는 것을 선택했다.

Kafka0.8에서  kafka0.10 으로 업데이트된건 나중에 따로 정리를 할것이다.

```xml
groupId = org.apache.spark
artifactId = spark-streaming-kafka-0-10_2.11
version = 2.2.0
```

gradle 

```groovy
implementation 'org.apache.spark:spark-streaming-kafka-0-10_2.11:2.4.3'
```

maven

```xml
<dependency>
  <groupId>org.apache.spark</groupId>
  <artifactId>spark-streaming-kafka-0-10_2.11</artifactId>
  <version>2.4.3</version>
</dependency>
```

https://spark.apache.org/docs/2.2.0/streaming-kafka-0-10-integration.html



spark 코드는 다음과 같다

```scala
package com.kafkastream.word

import org.apache.kafka.clients.consumer.ConsumerRecord
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.SparkConf
import org.apache.spark.streaming.dstream.InputDStream
import org.apache.spark.streaming.kafka010.ConsumerStrategies.Subscribe
import org.apache.spark.streaming.kafka010.KafkaUtils
import org.apache.spark.streaming.kafka010.LocationStrategies.PreferConsistent
import org.apache.spark.streaming.{Seconds, StreamingContext}

object KafkaStreaming{
  def main(args: Array[String]): Unit = {
    println("hello")
    val conf = new SparkConf().setAppName("KafkaSpark").setMaster("local")
    val ssc = new StreamingContext(conf, Seconds(10))

    val kafkaParams = Map(
      "bootstrap.servers" -> "localhost:9092",
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> "testGroup",
      "auto.offset.reset" -> "latest",
      "enable.auto.commit" -> (false: java.lang.Boolean))
    val topics = Set("search_word")
    val kafkaStream :InputDStream[ConsumerRecord[String, String]]= KafkaUtils.createDirectStream[String,String](ssc,PreferConsistent,Subscribe[String,String](topics,kafkaParams))

    kafkaStream.map(raw=>raw.value()).print()
    println("SparkContext refresh")
    ssc.start()
    ssc.awaitTermination()
  }
}

```

위와 같이 시작을 하고 producer에 `가위` 라는 값을 보내면 아래와 같이 에러가 난다 .

Serialization 관련 에러인 것같은데....

serialized key의 값이 -1이기 때문에  발생하는 에러인것같다.



key값을 출력하지않고 stream에서 value만 가지고 와서 출력을 하면 된다.



```scala
val kafkaStream :InputDStream[ConsumerRecord[String, String]]= KafkaUtils.createDirectStream[String,String](ssc,PreferConsistent,Subscribe[String,String](topics,kafkaParams))

kafkaStream.map(raw=>raw.value()).print() //추가한부분 value

ssc.start()
```



그리고 다시 실행한뒤 console-producer를 통해 `화장품` , `가방` 이라는 메세지를 보냈다. 

```scala
19/08/16 07:44:50 INFO TaskSchedulerImpl: Removed TaskSet 2.0, whose tasks have all completed, from pool 
19/08/16 07:44:50 INFO DAGScheduler: ResultStage 2 (print at KafkaStreaming.scala:28) finished in 0.080 s
19/08/16 07:44:50 INFO DAGScheduler: Job 2 finished: print at KafkaStreaming.scala:28, took 0.086579 s
-------------------------------------------
Time: 1565995490000 ms
-------------------------------------------
화장품
가방

19/08/16 07:44:50 INFO JobScheduler: Finished job streaming job 1565995490000 ms.0 from job set of time 1565995490000 ms
19/08/16 07:44:50 INFO MapPartitionsRDD: Removing RDD 3 from persistence list
19/08/16 07:44:50 INFO JobScheduler: Total delay: 0.107 s for time 1565995490000 ms (execution: 0.098 s)
19/08/16 07:44:50 INFO BlockManager: Removing RDD 3
```



https://www.rittmanmead.com/blog/2019/03/analysing-kafka-data-in-scala-spark/



Jar 파일로 만들어서 테스트하기