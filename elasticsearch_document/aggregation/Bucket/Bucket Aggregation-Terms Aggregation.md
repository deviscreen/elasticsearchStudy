# Bucket Aggregation-Terms Aggregation

하나의 고유한 값으로 그룹화하고 고유한값을 가진 도큐먼트의 갯수의 버킷이 동적으로 만든다.

```sql
select catecory,count(*) from usageReport Group by category order by count(*) DESC;
```



### Terms Aggregation

복합 버켓의 소스에 기반한 집합 

```json
GET /_search
{
    "aggs" : {
        "genres" : {
            "terms" : { "field" : "genre" } //해당 필드는 keyword 타입이어야한다.
        }
    }
}
```

결과 값

```json
{
    ...
    "aggregations" : {
        "genres" : {
            "doc_count_error_upper_bound": 0, // 1
            "sum_other_doc_count": 0, //2
            "buckets" : [ 
                {
                    "key" : "electronic",
                    "doc_count" : 6
                },
                {
                    "key" : "rock",
                    "doc_count" : 3
                },
                {
                    "key" : "jazz",
                    "doc_count" : 2
                }
            ]
        }
    }
}
```

1 (doc_count_error_upper_bound) : 집계 수행시 발생한 오류 개수 , 

​	데이터는 각 샤드로 분할이 된다. 각 샤드가 모든 버킷 키에 대한 데이터를 보내면 네트워크를 통해 전송되는 데이터가 너무 많아진다. 그렇기 때문에 상위 N 개에 대한 집계가 요청될 경우, 네트워크를 통해  N개의 버킷만 전송한다.

2 sum_other_doc_count : 반환되는 버킷에 포함되지 않은 총 도큐먼트 개수

​	예를들어 10개 이상의 고유한 버킷이 있는 경우는 상위 10개의 버킷을 반환, 10개 버킷외에 나머지 도큐먼트는 합쳐져 해당 필드에 반환