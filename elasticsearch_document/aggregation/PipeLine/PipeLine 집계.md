# PipeLine Aggregation



다른 집계를 토대로 다시 집계할 수 있다.

집계 결과를 다른 집계 입력으로 보낼 수 있는 셈 

* ParentPipeline 집계 : 다른 집계 안에 Pipeline 집계를 중첩해 사용
* Sibling Pipeline 집계 : Pipeline이 완료된 원본 집계의 형제로 사용



시간경과에 따른 사용율 누적 합계 계산 

데이터 삽입

```json
PUT from_to/_doc/5
{
  "caseOID": 1,
  "from":"start",
  "to":"end",
  "worktime":123
}
```

전체 데이터 

```json
{
  "took": 10,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 5,
    "max_score": 1,
    "hits": [
      {
        "_index": "from_to",
        "_type": "_doc",
        "_id": "5",
        "_score": 1,
        "_source": {
          "caseOID": 1,
          "from": "start",
          "to": "end",
          "worktime": 123
        }
      },
      {
        "_index": "from_to",
        "_type": "_doc",
        "_id": "2",
        "_score": 1,
        "_source": {
          "caseOID": 2,
          "from": "start",
          "to": "end",
          "worktime": 123
        }
      },
      {
        "_index": "from_to",
        "_type": "_doc",
        "_id": "4",
        "_score": 1,
        "_source": {
          "caseOID": 1,
          "from": "start",
          "to": "end",
          "worktime": 123
        }
      },
      {
        "_index": "from_to",
        "_type": "_doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "caseOID": 2,
          "from": "start",
          "to": "end",
          "worktime": 123
        }
      },
      {
        "_index": "from_to",
        "_type": "_doc",
        "_id": "3",
        "_score": 1,
        "_source": {
          "caseOID": 1,
          "from": "start",
          "to": "end",
          "worktime": 123
        }
      }
    ]
  }
}
```





setp1 

```json
GET from_to/_search
{
  "query": {
    "match_all": {}
  },
  "size": 0,
  "aggs": {
    "test_term": {
      "terms": {
        "field": "caseOID"
      },
      "aggs": {
        "test_sum": {
          "sum": {
            "field": "worktime"
          }
        }
      }
    },
    "tt_stats": {
      "stats_bucket": {
        "buckets_path": "test_term>test_sum"
      }
    }
  }
}
```

결과

```json
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 5,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "test_term": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": 1,
          "doc_count": 3,
          "test_sum": {
            "value": 369
          }
        },
        {
          "key": 2,
          "doc_count": 2,
          "test_sum": {
            "value": 246
          }
        }
      ]
    }
  }
}
```



쿼리

```json
GET from_to/_search
{
  "query": {
    "match_all": {}
  },
  "size": 0,
  "aggs": {
    "test_term": {
      "terms": {
        "field": "caseOID"
      },
      "aggs": {
        "test_sum": {
          "sum": {
            "field": "worktime"
          }
        }
      },
      "tt_stats":{
        "stats_bucket": {
          "buckets_path":"test_term>test_sum"
        }
      }
    }
  }
}
```

여기에서 buckets_path의 값을 "test_tertm > test_sum"에 부등호가 들어갔는데, `>` 는 group을 하는 path 를 지정한다고 생각하는것 같다.

buckets_path를 쓰려면 무조건 마지막 path aggregation은 숫자 또는 수치형이 나와야한다.



쿼리 : buckets_path의 결과가 수치형이여야 하기 때문에, buckets_path를 버킷의 키로 설정하여 쿼리를 실행

```json
GET from_to/_search
{
  "query": {
    "match_all": {}
  },
  "size": 0,
  "aggs": {
    "test_term": {
      "terms": {
        "field": "caseOID"
      }
    },
    "distinct_count": {
      "stats_bucket": {
        "buckets_path": "test_term._key"
      }
    }
  }
}
```

결과 값

```json
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 5,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "test_term": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": 1,
          "doc_count": 3
        },
        {
          "key": 2,
          "doc_count": 2
        }
      ]
    },
    "distinct_count": {
      "count": 2,
      "min": 1,
      "max": 2,
      "avg": 1.5,
      "sum": 3
    }
  }
}
```



버킷의 결과가 너무 많을경우 부하가 많을수도있기 때문에 아래와 같이 필터 path를 통해서 필요한것만 받아온다.

```json
GET from_to/_search?filter_path=aggregations.distinct_count
{
  "query": {
    "match_all": {}
  },
  "size": 0,
  "aggs": {
    "test_term": {
      "terms": {
        "field": "caseOID"
      }
    },
    "distinct_count": {
      "stats_bucket": {
        "buckets_path": "test_term._key"
      }
    }
  }
}
```

