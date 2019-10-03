# Value Count Aggregation

matics aggregation 으로 집계된 다큐먼트의 값의 수를 출력한다.  중복제거가 안됨  



아래와 같은 쿼리를 사용하여  3개의 도큐먼트를  삽입

```json
PUT my_index/_doc/3
{
  "my_field":5
}
```

이후 하나는 아래와 같은 도큐먼트를 삽입 하였다.

```json
PUT my_index/_doc/4
{
  "my_field2":5
}
```



`GET my_index/_search?size=0` 에 대한 결과 이다

```json
{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 4,
    "max_score": 1,
    "hits": [
      {
        "_index": "my_index",
        "_type": "_doc",
        "_id": "2",
        "_score": 1,
        "_source": {
          "my_field": 4
        }
      },
      {
        "_index": "my_index",
        "_type": "_doc",
        "_id": "4",
        "_score": 1,
        "_source": {
          "my_field2": 5
        }
      },
      {
        "_index": "my_index",
        "_type": "_doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "my_field": 4
        }
      },
      {
        "_index": "my_index",
        "_type": "_doc",
        "_id": "3",
        "_score": 1,
        "_source": {
          "my_field": 5
        }
      }
    ]
  }
}
```





이후 value count aggregation을 사용해보면 

```json
POST my_index/_search?size=0
{
  "aggs":{
    "types_count":{
      "value_count": {"field":"my_field"}
    }
  }
}
```

결과는 다음과 같이 출력 된다.

```json
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 4,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "types_count": {
      "value": 3
    }
  }
}
```

해당 필드에 해당하는 값만을 가지고 count를 한다 