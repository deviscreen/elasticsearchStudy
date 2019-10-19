# Save Log & Top Query



로그 저장 할 realtime_word 인덱스 생성 및 맵핑 설정

*  데이터타입이 keyword로 저장되기 때문에 timestamp인 reqdt는 date 형식으로 지정
* 키워드도 text 타입이 아닌 keyword로 지정

```json
DELETE realtime_word
PUT realtime_word
{}
PUT realtime_word/_mapping/doc
{
  "properties":{
    "searchWd":{
      "type": "keyword"
    },
    "reqdt":{
      "type": "date"
    }
  }
}
```



로그 수집 할 logstash 스크립트

```yaml
input {
       file {
               path => “/home/centos/logs/SearchWd.log”
               start_position => “beginning”
               sincedb_path => “/dev/null”
               codec => plain {
                       charset => “UTF-8"
               }
       }
}
filter {
       grok {
               match => {“message” => “search\[%{DATA:searchWd}\]+ timestamp\[%{DATA:reqdt}\]+“}
       }
}
output{
       elasticsearch {
               hosts => [“127.0.0.1:9200”]
               index => “realtime_word”
       }
       stdout {}
}
```



테스트 쿼리 

```json
##데이터 조회
GET realtime_word/_search
{
  "query":{
    "match_all": {}
  },
  "size":2
}
##탑 5개의 쿼리 
GET realtime_word/_search
{  
  "size":0,
  "aggs":{
    "top_5_word":{
      "terms": {"field":"searchWd"}
    }
  }
}
```



검색된 로그에서 5분내에  인기 검색어 5개만 가지고오기

* gte :  요청한 시간대 - 시간(현재시간 시점으로 5분 전 )
* lte : 추후 요청시에 보내는 타임스탬프

```json
GET realtime_word/_search
{
  "query":{
    "range":{
      "reqdt":{
        "gte":20191019021220,
        "lte":20191019021245
      }
    }
  },
  "size":0,
  "aggs":{
    "top_5_word":{
      "terms": {"field":"searchWd"}
      
    }
  }
}
```

