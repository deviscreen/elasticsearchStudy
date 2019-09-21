#인기 검색어 순위 만들기 (Kafka Spark Dstreaming)-(4)

###Elasticsearch 에서 field의 고유값의 갯수 세기

이전 글에서 Spark를 통해  kafka에서  메시지를  Elasticsearch에 저장까지 완료 하였다.

이제 terms aggregation을 통해 필드의 갯수만 집계하면될것이라 생각했는데, 에러 발생 에러발생! 

```json
GET realtime_word/_search
{
  "query":{
   "match_all": {}
  },
  "size":0,
  "aggregations":{
    "count_word":{
      "terms":{
        "field":"word"
      }
    }
  }
}
```

아래와 같은 에러가...

```json
{
  "error": {
    "root_cause": [
      {
        "type": "illegal_argument_exception",
        "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [word] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead."
      }
    ],
    "type": "search_phase_execution_exception",
    "reason": "all shards failed",
    "phase": "query",
    "grouped": true,
    "failed_shards": [
      {
        "shard": 0,
        "index": "realtime_word",
        "node": "uCyuJNg-TVu3E7g7nzRAZw",
        "reason": {
          "type": "illegal_argument_exception",
          "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [word] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead."
        }
      }
    ],
    "caused_by": {
      "type": "illegal_argument_exception",
      "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [word] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead.",
      "caused_by": {
        "type": "illegal_argument_exception",
        "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [word] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead."
      }
    }
  },
  "status": 400
}
```

이유는 아래의 블로그 참고 했다.

기존의 맵핑 

```json
{
  "realtime_word" : {
    "mappings" : {
      "_doc" : {
        "properties" : {
          "word" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          }
        }
      }
    }
  }
}

```



기존의 맵핑 변경하기

(6.5버전을 사용하기 때문에 뒤에 _doc를 붙여줌)

```json
PUT realtime_word/_mapping/_doc
{
  "properties": {
    "word": { 
      "type":     "text",
      "fielddata": true
    }
  }
}
```



변경된 맵핑 정보

```json
{
  "realtime_word" : {
    "mappings" : {
      "_doc" : {
        "properties" : {
          "word" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            },
            "fielddata" : true
          }
        }
      }
    }
  }
}

```



다시 terms aggregation을 사용해서 결과를 얻어오게 되면 ? 아래와 같은 결과를 얻을수 있다.

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
    "total" : 7,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "count_word" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "samsung",
          "doc_count" : 1
        },
        {
          "key" : "시계",
          "doc_count" : 1
        },
        {
          "key" : "안경",
          "doc_count" : 1
        },
        {
          "key" : "의자",
          "doc_count" : 1
        },
        {
          "key" : "책상",
          "doc_count" : 1
        },
        {
          "key" : "컵",
          "doc_count" : 1
        },
        {
          "key" : "테이블",
          "doc_count" : 1
        }
      ]
    }
  }
}

```





---

참고

http://blog.naver.com/PostView.nhn?blogId=rokking1&logNo=221366675132&categoryNo=0&parentCategoryNo=30&viewDate=&currentPage=1&postListTopCurrentPage=1&from=search

https://www.elastic.co/guide/en/elasticsearch/reference/current/fielddata.html



