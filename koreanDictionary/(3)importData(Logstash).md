# 사전 만들기 프로젝트(3)-전처리된 데이터를 삽입해보자(logstash) 

logstash를 사용해서 데이터를 삽입했다



korean_dic.conf

```yaml
input {
  file {
    path => "//Users/daeyunkim/Documents/07.ELK_v6/data/korean_dic/filter_korean5.txt"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}
filter {
  csv {
    separator => "|"
    columns => ["voca","isNative","partOfSpeech","meaning","category"]
  }

  mutate {
    remove_field => ["@version","@timestamp","path","host", "tags", "message"]
  } 
}
output {
  elasticsearch {
    hosts => "http://localhost:9200"
    index => "dic_korean"
    document_type => "_doc"
  }
  stdout {}
}

```



```shell
$ $LOGSTASH_HOME/bin/logstash -f korean_dic.conf
```

위와 같이 실행을 하게 되면 

데이터가 들어가는 것을 확인할수 있다.

확인을 해보자!! 

```json
GET dic_korean/_search
{
  "query":{
    "match_all":{
      
    }
  },"size":3
}
```

3개만 불러와보자 

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 30002,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "dic_korean",
        "_type" : "_doc",
        "_id" : "KDPEoGsBaqdVh3ql3lt9",
        "_score" : 1.0,
        "_source" : {
          "partOfSpeech" : "「명사」",
          "voca" : "가가-성자",
          "category" : "일반어",
          "meaning" : "성문 사과(聲聞四果)에서, 일래과를 얻기 위하여 수행하는 성인.",
          "isNative" : "한자어"
        }
      },
      {
        "_index" : "dic_korean",
        "_type" : "_doc",
        "_id" : "LTPEoGsBaqdVh3ql3lt9",
        "_score" : 1.0,
        "_source" : {
          "partOfSpeech" : "「부사」",
          "voca" : "가강-히",
          "category" : "일반어",
          "meaning" : "더욱 강력하고 완강하게.",
          "isNative" : "혼종어"
        }
      },
      {
        "_index" : "dic_korean",
        "_type" : "_doc",
        "_id" : "MDPEoGsBaqdVh3ql3lt9",
        "_score" : 1.0,
        "_source" : {
          "partOfSpeech" : "「명사」",
          "voca" : "가겟-세",
          "category" : "일반어",
          "meaning" : "가게를 하는 자리를 빌려 쓰는 대가로 주는 돈.",
          "isNative" : "혼종어"
        }
      }
    ]
  }
}

```

데이터가 제대로 들어간것을 알수 있다.

이번주 목표 끝~~!!