# 인기 검색어 순위 만들기 (Kafka Spark Dstreaming)-(3)

### Spark Consumer로 받은 데이터를 Elasticsearch에 저장하기 

dstream 과 elasticsearch의 spark context 두개가 있는데, spark에서는 하나의 jvm에서 하나의 context만 만들수 있다. 



공식 문서를 참조하여 아래 처럼 SparkContext객체를 만들어서 streamingContext의 인자 값으로 넣어준다.

```scala
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._     //1          
import org.apache.spark.streaming.StreamingContext
import org.apache.spark.streaming.StreamingContext._

import org.elasticsearch.spark.streaming._   //2   

...

val conf = ...
val sc = new SparkContext(conf)             //3         
val ssc = new StreamingContext(sc, Seconds(1))  //4     

val numbers = Map("one" -> 1, "two" -> 2, "three" -> 3)
val airports = Map("arrival" -> "Otopeni", "SFO" -> "San Fran")

val rdd = sc.makeRDD(Seq(numbers, airports))
val microbatches = mutable.Queue(rdd)       //5         

ssc.queueStream(microbatches).saveToEs("spark/docs") //6

ssc.start()
ssc.awaitTermination() //7
```

아래의 설명

```txt
1. Spark and Spark Streaming Scala imports

2. elasticsearch-hadoop Spark Streaming imports

3. start Spark through its Scala API

4. start SparkStreaming context by passing it the SparkContext. The microbatches will be processed every second.

5. makeRDD creates an ad-hoc RDD based on the collection specified; any other RDD (in Java or Scala) can be passed in. Create a queue of `RDD`s to signify the microbatches to perform.

6. Create a DStream out of the RDD`s and index the content (namely the two _documents_ (numbers and airports)) in {es} under `spark/docs

7. Start the spark Streaming Job and wait for it to eventually finish.
```



Elasticsearch-Spark 공식 문서 : https://www.elastic.co/guide/en/elasticsearch/hadoop/master/spark.html

위의 문서를 응용하여 저장하는 아래의 코드를 추가 하였다.

```scala
		//추가 한 부분 
    val streamData  =  kafkaStream.map(raw=>Map("word"->raw.value()))
    println("SparkContext refresh")
    streamData.print()
    streamData.saveToEs("realtime_word/_doc")
```

spark streaming을 이용하여 Elasticsearch에 저장하는 최종 코드는 아래와 같이 완성이 되었다.

```scala
package com.kafkastream.word

import org.apache.kafka.clients.consumer.ConsumerRecord
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.streaming.dstream.InputDStream
import org.apache.spark.streaming.kafka010.ConsumerStrategies.Subscribe
import org.apache.spark.streaming.kafka010.KafkaUtils
import org.apache.spark.streaming.kafka010.LocationStrategies.PreferConsistent
import org.apache.spark.streaming.{Seconds, StreamingContext}
import org.apache.spark.streaming.StreamingContext._
import org.elasticsearch.spark.streaming._

import scala.collection.mutable
object KafkaStreaming{
  def main(args: Array[String]): Unit = {
    println("hello")
    val conf = new SparkConf().setAppName("KafkaSpark").setMaster("local")
    conf.set("es.index.auto.create","true")
    conf.set("es.nodes","127.0.0.1")
    conf.set("es.port","9200")
    val sc = new SparkContext(conf)
    val ssc = new StreamingContext(sc, Seconds(10))//stream context

    val kafkaParams = Map(
      "bootstrap.servers" -> "localhost:9092",
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> "testGroup",
      "auto.offset.reset" -> "latest",
      "enable.auto.commit" -> (false: java.lang.Boolean))
    val topics = Set("search_word")
    val kafkaStream :InputDStream[ConsumerRecord[String, String]]= KafkaUtils.createDirectStream[String,String](ssc,PreferConsistent,Subscribe[String,String](topics,kafkaParams))
    
		//추가 한 부분 
    val streamData  =  kafkaStream.map(raw=>Map("word"->raw.value()))
    println("SparkContext refresh")
    streamData.print()
    streamData.saveToEs("realtime_word/_doc")

    ssc.start()
    ssc.awaitTermination()
  }
}
```



elasticsearch를 통해 테스트를 해보자 

kafka-console-producer.sh 를 통해 아래와 같은 메시지를 생성하였다.

```shell
>의자
>시계
>컵
```



spark streaming 로그

```txt
19/08/17 11:14:20 INFO DAGScheduler: ResultStage 24 (print at KafkaStreaming.scala:40) finished in 0.009 s
19/08/17 11:14:20 INFO DAGScheduler: Job 24 finished: print at KafkaStreaming.scala:40, took 0.010485 s
-------------------------------------------
Time: 1566008060000 ms
-------------------------------------------
Map(word -> 의자 )

19/08/17 11:14:20 INFO JobScheduler: Finished job streaming job 1566008060000 ms.0 from job set of time 1566008060000 ms
19/08/17 11:14:20 INFO JobScheduler: Starting job streaming job 1566008060000 ms.1 from job set of time 1566008060000 ms
....
19/08/17 11:14:30 INFO DAGScheduler: ResultStage 26 (print at KafkaStreaming.scala:40) finished in 0.009 s
19/08/17 11:14:30 INFO DAGScheduler: Job 26 finished: print at KafkaStreaming.scala:40, took 0.012619 s
-------------------------------------------
Time: 1566008070000 ms
-------------------------------------------
Map(word -> 시계)
Map(word -> 컵)
```





elasticsearch 에서 `realtime_word` 의 인덱스에 doc가 제대로 들어갔는지 확인

(kibana에서 확인)

```json
GET realtime_word/_search
```

이전에 테스트로 들어가있는 단어 외에 producer를 통해 추가가 된것을 알수 있다.

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 7,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "realtime_word",
        "_type" : "_doc",
        "_id" : "Q8FZnWwBwztwv-DDmfgE",
        "_score" : 1.0,
        "_source" : {
          "word" : "테이블"
        }
      },
      {
        "_index" : "realtime_word",
        "_type" : "_doc",
        "_id" : "lsFanWwBwztwv-DD0fiA",
        "_score" : 1.0,
        "_source" : {
          "word" : "시계"
        }
      },
      {
        "_index" : "realtime_word",
        "_type" : "_doc",
        "_id" : "RMFZnWwBwztwv-DDmfgE",
        "_score" : 1.0,
        "_source" : {
          "word" : "책상"
        }
      },
      {
        "_index" : "realtime_word",
        "_type" : "_doc",
        "_id" : "i8FanWwBwztwv-DDqvhu",
        "_score" : 1.0,
        "_source" : {
          "word" : "의자 "
        }
      },
      {
        "_index" : "realtime_word",
        "_type" : "_doc",
        "_id" : "McFNnWwBwztwv-DD_vXD",
        "_score" : 1.0,
        "_source" : {
          "word" : "samsung"
        }
      },
      {
        "_index" : "realtime_word",
        "_type" : "_doc",
        "_id" : "OMFZnWwBwztwv-DDcvgy",
        "_score" : 1.0,
        "_source" : {
          "word" : "안경"
        }
      },
      {
        "_index" : "realtime_word",
        "_type" : "_doc",
        "_id" : "l8FanWwBwztwv-DD0fiA",
        "_score" : 1.0,
        "_source" : {
          "word" : "컵"
        }
      }
    ]
  }
}
```



