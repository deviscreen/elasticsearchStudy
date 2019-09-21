# 인기 검색어 순위 만들기 (Kafka Spark Dstreaming)-(1)

웹에서 단어 검색 어플리케이션을 작성 하였으면 이제는 단어를 검색한 결과의 로그를 사용하여 인기 검색어를 만들어보기로 하였다.

작업이 두가지로 나뉘어지는데, 첫번째는 spring log에서 kafka로 검색로그를 보내기, 두번째는 kafka에 쌓인 데이터를 spark에서 5분마다 elasticsearch 데이터에 삽입하기 

내가 맡은 부분은 spark로 kafka 데이터를 subscribe 해서 5분마다 elasticsearch에 넣어주는 모듈을 만들기로 했다.

```txt
kafka consumer를 통해서 spark streaming
Kafka brocker => spark streaming
(시간, 단어) => 1분동안 데이터 모아서 streaming 처리 

Kafka consumer에서 위와 같은 데이터 불러와서 spark streaming으로 받아오기
```

작업 순서

1. Kafka 설치 
2. producer  에서 데이터 보내기 
3. spark streaming (consumer 연결하기)
4. es에 저장하기 

작업해야할것 

###1. Kafka 설치 후 producer/consumer 테스트

카프카 홈페이지에서 다운로드 후 설치 

Kafka 를 실행 하기 위해서는 zookeeper가 필수로 필요하지만

kafka 파일안에 단일노드에서 실행하기 위한 zookeeper 실행파일, config 파일이 포함되어있다.

( kafka_home/bin/zookeeper-server-start.sh, kafka_home/config/zookeeper.properties )



(1). zookeeper 실행

```shell
$cd Kafka_HOME
$./bin/zookeeper-server-start.sh ./config/zookeeper.properties
[2019-08-15 12:37:18,846] INFO Reading configuration from: ./config/zookeeper.properties (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
.....
[2019-08-15 12:37:18,907] INFO Using org.apache.zookeeper.server.NIOServerCnxnFactory as server connection factory (org.apache.zookeeper.server.ServerCnxnFactory)
[2019-08-15 12:37:18,923] INFO binding to port 0.0.0.0/0.0.0.0:2181 (org.apache.zookeeper.server.NIOServerCnxnFactory)
```



(2). kafka 실행

```shell
$./bin/kafka-server-start.sh ./config/server.properties
[2019-08-15 12:39:31,139] INFO Registered kafka:type=kafka.Log4jController MBean (kafka.utils.Log4jControllerRegistration$)
...
[2019-08-15 12:39:32,253] INFO Kafka startTimeMs: 1565840372251 (org.apache.kafka.common.utils.AppInfoParser)
[2019-08-15 12:39:32,254] INFO [KafkaServer id=0] started (kafka.server.KafkaServer)
```



(3). 프로세스 확인

```shell
$jps
98868 GradleDaemon
5206 QuorumPeerMain
5639 Kafka
10906
5995 Jps
```

`5206 QuorumPeerMain` 은 zookeeper,`5639 Kafka` 는 Kafka 를 의미

Kafka는 토픽을  통해 producer와 consumer로 데이터를 생산하고 받아진다.



(4). 토픽 생성

단어 검색의 로그 결과를 받는 토픽은 `search_word` 로 설정 

```shell
##토픽 생성
$./bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic search_word
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.

##토픽 생성 확인
$./bin/kafka-topics.sh --list --bootstrap-server localhost:9092
search_word
```



​	토픽이 생성되면 producer에 매세지를 보내고 consumer를 통해 받는 테스트를 해보겠다.



​	(5). 메시지 보내기(console-producer)

```shell
$./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic search_word
>hi
>가위
```

​	(6). 메시지 받기(console-consumer)	

```shell
$./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic search_word --from-beginning
hi
가위
```

console을 통해 확인을 한 결과 제대로 설치가 된 것 같다.

 더 다양한 옵션들도 지원한다.

