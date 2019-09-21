# 인기 검색어 순위 만들기 (Kafka Spark Dstreaming)-(5) 



위에서 spark의 코드를작성하고 테스트 까지 완료 하였다.

이제 배포를 해야하는데 어떻게 할까 ?

참고 

https://github.com/faizanahemad/spark-gradle-template/blob/master/build.gradle

실행 가능한 fat jar로 만들기 위해서 하위 부분  jar 를 추가 해주었다.

```gradle
plugins {
    id 'scala'

}

group 'KafkaEsSpark'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
    // https://mvnrepository.com/artifact/org.scala-lang/scala-library
    compile group: 'org.scala-lang', name: 'scala-library', version: '2.11.7'
    compile 'org.apache.spark:spark-core_2.11:2.4.3'
    compile 'com.crealytics:spark-excel_2.11:0.12.0'
    compile 'org.apache.spark:spark-sql_2.11:2.4.3'
    compile 'org.elasticsearch:elasticsearch-spark-20_2.11:6.5.2'
    // https://mvnrepository.com/artifact/org.apache.spark/spark-streaming
    compile 'org.apache.spark:spark-streaming_2.11:2.4.3'
    compile 'org.apache.spark:spark-streaming-kafka-0-10_2.11:2.4.3'
}

jar {
    classifier = 'all'
    manifest {
        attributes "Main-Class": "com.kafkastream.word.KafkaStreaming"
    }
    from{
        configurations.compile.collect { it.isDirectory() ? it: zipTree(it)}
    }
    baseName = 'SparkEsKafka'
    zip64 true
}

task run(type:JavaExec, dependsOn: classes) {
    main = "com.kafkastream.word.KafkaStreaming"
    classpath sourceSets.main.runtimeClasspath
    classpath configurations.runtime
}
task fatJar(type: Jar){
    zip64 true
    description = "Assembles a Hadoop ready fat jar file"
    baseName = project.name + '-all'
    doFirst {
        from {
            configurations.compile.collect { it.isDirectory() ? it : zipTree(it) }
        }
    }
    manifest {
        attributes "Main-Class": "com.kafkastream.word.KafkaStreaming"
    }
    exclude 'META-INF/*.RSA','META-INF/*.SF','META-INF/*.DSA'
    with jar
}
```



프로젝트 루트 디렉토리에있는 gradlew 파일이 있는 곳에서 

```shell
$./gradlew fatJar
```

를 하게 되면 프로젝트 루트폴더 build> libs 에 kafkaspark-all-1.0-SNAPSHOT.jar 파일이 보인다.

```shell
/kafkaspark/build/libs$ls

SparkEsKafka-1.0-SNAPSHOT-all.jar 
kafkaspark-all-1.0-SNAPSHOT.jar
```



모든 라이브러리들의 의존성을 한번에 묶어줬기 때문에 spark-submit을 통해 제출하지 않아도 실행이 되는것같다.

```shell
$java -jar kafkaspark-all-1.0-SNAPSHOT.jar
```



실행을 해보면 ?

```shell
/build/libs$java -jar kafkaspark-all-1.0-SNAPSHOT.jar 
hello
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
19/08/24 12:02:40 INFO SparkContext: Running Spark version 2.4.3
19/08/24 12:02:40 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
19/08/24 12:02:40 INFO SparkContext: Submitted application: KafkaSpark
19/08/24 12:02:40 INFO SecurityManager: Changing view acls to: daeyunkim
19/08/24 12:02:40 INFO SecurityManager: Changing modify acls to: daeyunkim
19/08/24 12:02:40 INFO SecurityManager: Changing view acls groups to: 
19/08/24 12:02:40 INFO SecurityManager: Changing modify acls groups to: 
19/08/24 12:02:40 INFO SecurityManager: SecurityManager: authentication disabled;
...
19/08/24 12:02:41 WARN StreamingContext: spark.master should be set as local[n], n > 1 in local mode if you have receivers to get data, otherwise Spark jobs will not get resources to process the received data.
19/08/24 12:02:41 WARN KafkaUtils: overriding enable.auto.commit to false for executor
19/08/24 12:02:41 WARN KafkaUtils: overriding auto.offset.reset to none for executor
19/08/24 12:02:41 WARN KafkaUtils: overriding executor group.id to spark-executor-testGroup
19/08/24 12:02:41 WARN KafkaUtils: overriding receive.buffer.bytes to 65536 see KAFKA-3135
....
19/08/24 12:02:41 INFO MappedDStream: Initialized and validated 
-------------------------------------------
Time: 1566615780000 ms
-------------------------------------------
...
```

실행이 잘된다 .

실행한 상태에서 

search_word에 토픽에 전달을 하게 되면 잘 받아와서 elasticsearch에  잘 저장됨



소스는 여기에서 볼수 있다. (약간 수정해서 kafka broker의 아이디와 elasticsearch의 주소를 적을수 있게 변경했다.)

https://github.com/DaeyunKim/Kafka-SparkStreaming-Elasticsearch



실행 순서 : 

1. kafka 를 위한 zookeeper 실행 

2. Kafka 서버 실행 - 토픽 (search_word) 추가 

3. elasticsearch 실행 

4. Build 한 jar 파일 만들기 

   ProjectRoot에서 실행 

   ```shell
   $./gradlew fatJar
   ```

5. (테스트) kafka console producer를 통해서 테스트 실행 함 

   kafkaspark-all-1.0-SNAPSHOT.jar 는 프로젝트 루트의 아래 /build/lib에 위치해있다.

   ```shell
   $java -jar kafkaspark-all-1.0-SNAPSHOT.jar 127.0.0.1 9200 localhost:9092
   ```



spark streaming 으로 kafka-0.10 consumer api를 사용하여 elasticsearch에 저장하는 프로젝트



참고자료

http://www.hongyusu.com/amt/spark-streaming-kafka-avro-and-registry.html


