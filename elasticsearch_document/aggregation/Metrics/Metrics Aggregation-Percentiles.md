# Metrics Aggregation Percentiles

근사치 통계

Percentiles Aggregation

집계된 document 로 부터 하나또는 그 이상의 퍼센트의  숫치 값 을 추출,

이러한 수치들은 document에서 명확한 숫자 필드로부터 추출되거나, 제공된 스크립트에 의해 만들어진다.



주어진 X를 가지고 전체 값에서 x% 이하인 값을 찾는다.

온라인 상점을 운영하고 있다면, 각 장바구니에 있는 값을 기록하고, 사용자들이 어떤 가격 범위를 장바구니에 담는지 확인하기 유용.

정확도는 0 또는 100으로 향할수록 더 좋은 결과를 얻는다.

Percentiles는 주어진 X를 가지고 전체 값에서 X% 이하인 값을 찾는다.



percentiles 는 보통 대략적으로 나타낸다.

많은 알고리즘이 있지만, 기본적인 것은 저장소에서 배열을 정렬하고, 50%의 값을 찾는 값이다.

`my_array[count(my_array) * 0.5]`

전체 데이터를 sort 하는 것은 데이터가 많을수록 많은 선형적으로 늘어나게 된다. 

[Computing Accurate Quantiles using ](https://github.com/tdunning/t-digest/blob/master/docs/t-digest-paper/histo.pdf)[T-Digests](https://github.com/tdunning/t-digest/blob/master/docs/t-digest-paper/histo.pdf)  알고리즘을 쓰는 이유는 정확하게 하려면 전체 값을 가지고 와서 정렬 후 중간 값을 가지고 와야하지만 전체 정렬은 데이터가 커질수록 부담이 커지는 단점 을 보완하기 위해서 사용하는 것 같다.

잠재적인 값들을 수십억개의 값에 걸쳐 백분위수를 계한하기 위해 대략적인 백분위 수를 계산한다. elasticsearch cluster 에서 대략적으로 나타낸다.



 예제 데이터

```json
POST attendcount/_doc/1
{
  "persons": 5
}
POST attendcount/_doc/2
{
  "persons": 4
}
POST attendcount/_doc/3
{
  "persons": 3
}
POST attendcount/_doc/4
{
  "persons": 2
}
POST attendcount/_doc/5
{
  "persons": 1
}
```

쿼리

```query
GET attendcount/_doc/_search
{
  "size":0,
  "aggs": {
    "attendees_percentiles": {
      "percentiles": {
        "field": "persons",
        "percents": [
          10,50,100
        ]
      }
    }
  }
}
```

결과

```json
{
  "took": 7,
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
    "attendees_percentiles": {
      "values": {
        "10.0": 1.4,
        "50.0": 3,
        "100.0": 5
      }
    }
  }
}
```





percentile 구하는 과정 

데이터셋: `188, 168, 174, 198, 186, 170, 180, 182, 186, 176`

데이터셋에서 20,80의 percentile을 찾는 방법 

1. 정렬 

   `168, 170, 174, 176, 180, 182, 186, 186, 188, 198`

2. 20th Percentile  =>  0.20 ( 10개) = 2 위치 찾기 

   `168, 170, 174....`

   168 ( 1 th)

   170 ( 2 th)

   174  (3 th)

   **결과 : 174**

3. 80th  Percentile => 0.80(10개) = 8 위치 찾기

   168 ( 1 th)

   170 ( 2 th)

   174 ( 3 th)

   176 ( 4 th)

   180 ( 5 th)

   182 ( 6 th)

   186 ( 7 th)

   186 ( 8 th)

   188 ( 9th)

   198 ( 10 th)

   **결과 : 188**

   몇 퍼센트의 숫자가 어떤 수의 이하에( below )있는지를 검사 하는것 ! 

   해당 값 이하의 데이터기 전체의 p%인 값을 의미 !! 

참고: <https://www.youtube.com/watch?v=MSQpvuPL2cw>



공식
$$
L_p = (P/100 )(n+1)
$$
p : 퍼센트

n : 갯 수 

예를들면 갯수가 15개이고 ,70퍼센타일을 를 찾는다고 하면 

0. 70( 15+1) => 11.2 

11.2 의 위치에 있는 값이 되는데 이것은 11과 12 번째 사이에 있다. 

이럴 경우에는 12번째의 수 + (12번쨰 값 - 11번쨰 값) * 0.5 의 값으로 한다.





기존 공식 

```python
def median(data):
    sorted_data = sorted(data)
    data_len = len(sorted_data)
    return 0.5*( sorted_data[(data_len-1)//2] + sorted_data[data_len//2])
```

쿼리 테스트 

```json
DELETE attendcount
POST attendcount/_doc/1
{
  "persons": 5
}
POST attendcount/_doc/2
{
  "persons": 4
}
POST attendcount/_doc/3
{
  "persons": 3
}

POST attendcount/_doc/4
{
  "persons": 2
}
POST attendcount/_doc/5
{
  "persons": 1
}
GET attendcount/_search

#exist percentiles
GET attendcount/_doc/_search
{
  "size":0,
  "aggs": {
    "attendees_percentiles": {
      "percentiles": {
        "field": "persons",
        "percents": [
          10,50,100
        ]
      }
    }
  }
}
GET attendcount/_search

##GET DOCUMENT COUNT
GET attendcount/_search?filter_path=_shards.total



##sort
GET attendcount/_doc/_search?filter_path=_shards.total,hits.hits._source, aggregations
{
  "query":{
    "match_all":{
    }
  },
  "sort":[
    { "persons" : {"order":"asc"}}  
  ],
  "aggs":{
    "max_median":{
      "max": {
        "field": "persons"
      }
    }
  }
}
##get median top query
GET attendcount/_doc/_search
{
  "query": {
    "match_all": {}
  },
  "size": 0,
  "aggs": {
    "median_persons": {
      "top_hits": {
        "sort": {
          "persons": {
            "order": "asc"
          }
        },
        "size": 3
      }
    }
  }
}

##get median top query
GET attendcount/_doc/_search
{
  "query": {
    "match_all": {}
  },
  "sort":[
    { "persons" : {"order":"asc"}}  
  ],
  "size":3
}
GET attendcount/_doc/_search
{
  "query":{
    "match_all":{}
  },
  "sort":{
    "persons":{"order":"asc"}
  },
  "aggs":{
    "statistic_person":{
      "stats":{
        "field":"persons"
      }
    }
  }
}

##Percentile Test
DELETE murderer
POST _bulk
{ "index":{ "_index":"murderer","_type":"_doc","_id":1}}
{ "point":2.4 }
{ "index":{ "_index":"murderer","_type":"_doc","_id":2}}
{ "point":2.8 }
{ "index":{ "_index":"murderer","_type":"_doc","_id":3}}
{ "point":4.4 }
{ "index":{ "_index":"murderer","_type":"_doc","_id":4}}
{ "point":4.7 }
{ "index":{ "_index":"murderer","_type":"_doc","_id":5}}
{ "point":5.6 }
{ "index":{ "_index":"murderer","_type":"_doc","_id":6}}
{ "point":5.6 }
{ "index":{ "_index":"murderer","_type":"_doc","_id":7}}
{ "point":5.7 }
{ "index":{ "_index":"murderer","_type":"_doc","_id":8}}
{ "point":5.8 }

GET murderer/_search

GET murderer/_search
{
  "query":{
    "match_all":{}
  }
  ,"size":0
  ,"aggs":{
    "percent_test":{
      "percentiles":{
        "field":"point"
        , "percents": [
          5,95
        ]
      }
    }
  }
}

DELETE bucket

POST _bulk
{"index":{"_index":"bucket","_type":"_doc","_id":1}}
{"point":168}
{"index":{"_index":"bucket","_type":"_doc","_id":2}}
{"point":170}
{"index":{"_index":"bucket","_type":"_doc","_id":3}}
{"point":174}
{"index":{"_index":"bucket","_type":"_doc","_id":4}}
{"point":176}
{"index":{"_index":"bucket","_type":"_doc","_id":5}}
{"point":180}
{"index":{"_index":"bucket","_type":"_doc","_id":6}}
{"point":182}
{"index":{"_index":"bucket","_type":"_doc","_id":7}}
{"point":186}
{"index":{"_index":"bucket","_type":"_doc","_id":8}}
{"point":186}
{"index":{"_index":"bucket","_type":"_doc","_id":9}}
{"point":188}
{"index":{"_index":"bucket","_type":"_doc","_id":10}}
{"point":198}

GET bucket/_search
{
  "query":{
    "match_all":{}
  }
  ,"size":0
  ,"aggs":{
    "percent_test":{
      "percentiles":{
        "field":"point"
        , "percents": [
          20,80
        ],
        "tdigest":{
          "compression":200
        }
      }
    }
  }
}
```

