#Composite Query(복합 쿼리)



다른 소스에서복합 버킷을 만드는 다중 버킷 집계,

어그리게이션 결과를 size 개수만큼 순차적으로 보여주는 기능 



다른 멀티 버킷 aggregation을 paginate 하게 하여 모든 버킷을 불러오는 것과 같음.



**예제 document**

```json
{
    "keyword": ["foo", "bar"],
    "number": [23, 65, 76]
}
```



composite 버킷은 키워드와 숫자 를 aggregation의 숫자로 쓰일때 사용된다.

```json
{ "keyword": "foo", "number": 23 }
{ "keyword": "foo", "number": 65 }
{ "keyword": "foo", "number": 76 }
{ "keyword": "bar", "number": 23 }
{ "keyword": "bar", "number": 65 }
{ "keyword": "bar", "number": 76 }
```



### Terms

terms 소스 값은  단순 term aggregation과 동등하다.

그  값은 필드로 부터 추출하거나 script 를 사용하여  terms aggregation을 수행한다.

예제( field )

```json
GET /_search
{
    "aggs" : {
        "my_buckets": {
            "composite" : {
                "sources" : [
                    { "product": { "terms" : { "field": "product" } } }
                ]
            }
        }
     }
}
```



예제( script)

```json
GET /_search
{
    "aggs" : {
        "my_buckets": {
            "composite" : {
                "sources" : [
                    {
                        "product": {
                            "terms" : {
                                "script" : {
                                    "source": "doc['product'].value",
                                    "lang": "painless"
                                }
                            }
                        }
                    }
                ]
            }
        }
    }
}
```



###실습

 쿼리

```json
GET csv_1juros0vr00hh/_search
{
  "query": {
    "match_all": {}
  },
  "size":0,
  "aggs": {
    "groupbyResourceoid": {
      "terms": {
        "field": "RESOURCEOID.keyword",
        "size":10020
      }
    }
  }
}
```

결과

```json
#! Deprecation: This aggregation creates too many buckets (10001) and will throw an error in future versions. You should update the [search.max_buckets] cluster setting or use the [composite] aggregation to paginate all buckets in multiple requests.
{
  "took": 376,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 6887446,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "groupbyResourceoid": {
      "doc_count_error_upper_bound": 5,
      "sum_other_doc_count": 51470,
      "buckets": [
        {
          "key": "TTL",
          "doc_count": 896253
        },....생략
```





###composite 쿼리 반영

```json
GET csv_1juros0vr00hh/_search
{
  "query": {
    "match_all": {}
  },
  "size":0,
  "aggs": {
    "groupbyResourceoid": {
      "composite": {
        "size":10000,
        "sources": [
          {
            "RESOURCEOID.keyword": {
              "terms": {
                "script":{
                  "source": "doc['RESOURCEOID.keyword'].value",
                  "lang": "painless"
                }
              }
            }
          }
        ]
      }
    }
  }
}
```



결과

```json
{
  "took": 12102,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 6887446,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "groupbyResourceoid": {
      "buckets": [
        {
          "key": {
            "RESOURCEOID.keyword": ""
          },
          "doc_count": 187753
        },
        {
          "key": {
            "RESOURCEOID.keyword": "H0000"
          },
          "doc_count": 41274
        },
        {
          "key": {
            "RESOURCEOID.keyword": "H0011"
          },
          "doc_count": 13
        },...생략(이전의 기본 쿼리 보다 더 많은 결과 )
```

