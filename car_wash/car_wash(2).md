# 주변 세차장 찾기 (2)

현재 위치를 중심으로 쿼리 만들기

현재 위치는 다양한맵의 주소에서 찾을수 있다.

구글 맵에서 검색을 하게 되면 쿼리 파라미터에 위도경도가 나오는것을 알수 있다. 

```txt
https://www.google.co.kr/maps/place/%EC%8B%A0%EB%8F%84%EB%A6%BC%EC%97%AD/@37.5088099,126.8890174,17z/data=!3m1!4b1!4m5!3m4!1s0x357c9e5dbb76b179:0x2f88a2d1152886e2!8m2!3d37.5088099!4d126.8912061?hl=ko
```

위도 경도를 찾아서 다음과 같은 정보를 얻을수 있다.

이제 쿼리를 만들어보자

위치를 사용한 쿼리는 아래와같이 4가지의 종류가 있다.

* Geo_shape 

  * Intersects, disjoint, within, contains 등의 모양에 맞게 검색
  * [Geoshape document](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/query-dsl-geo-shape-query.html)

* Geo_bounding_box

  * 사각형의 범위를 가지며 왼쪽 상단, 오른쪽 하단의 위도 경도를 사용하여 범위안의 값을 찾음

    [bounding_box docuemnt](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/query-dsl-geo-bounding-box-query.html) 

* Geo_distance

  * 특정 위도 경도와의 거리를 사용하여 특정 거리안의 범위값을 찾음

    [distance document](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/query-dsl-geo-distance-query.html)

* Geo_polygon

  * 특정 지점들을 연결연결 하여 다각형을 그리고 그 구역내의 범위를 찾음

    [polygon document ](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/query-dsl-geo-polygon-query.html)

스터디를 하는곳인 신도림역 37.5088099,126.8890174 을 중심으로 10Km 안에 주차장이 어디에 있는지 검색을 해봤다.

```json
GET car_wash/_search
{
  "query": {
        "bool" : {
            "must" : {
                "match_all" : {}
            },
            "filter" : {
                "geo_distance" : {
                    "distance" : "10km",
                    "location": {
                        "lat" : 37.5088099,
                        "lon" : 126.8890174 
                    }
                }
            }
        }
    },"size":5
}
```

결과값

```json
{
  "took" : 8,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 584,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "car_wash",
        "_type" : "_doc",
        "_id" : "I_HOcW4BGQw9X5k0OQxe",
        "_score" : 1.0,
        "_source" : {
          "privider_name" : "3170000",
          "column12" : "서울특별시 금천구",
          "name" : "화성자동차정비공업사",
          "state" : "서울특별시",
          "location" : "37.460736,126.898655",
          "water_quality" : "433",
          "sector" : "정비업소",
          "address" : "서울특별시 금천구 시흥대로 304 (독산동)",
          "phone" : "02-3286-8572",
          "city" : "금천구",
          "date,provider_id" : "2019.9.10"
        }
      },
      {
        "_index" : "car_wash",
        "_type" : "_doc",
        "_id" : "J_HOcW4BGQw9X5k0OQxe",
        "_score" : 1.0,
        "_source" : {
          "privider_name" : "3170000",
          "column12" : "서울특별시 금천구",
          "name" : "금천자동차검사정비센타",
          "state" : "서울특별시",
          "location" : "37.441008,126.902064",
          "water_quality" : "559",
          "sector" : "정비업소",
          "address" : "서울특별시 금천구 시흥대로27길 26 (시흥동)",
          "phone" : "02-893-5500",
          "city" : "금천구",
          "date,provider_id" : "2019.9.10"
        }
      },
      {
        "_index" : "car_wash",
        "_type" : "_doc",
        "_id" : "n_HOcW4BGQw9X5k0OwxT",
        "_score" : 1.0,
        "_source" : {
          "privider_name" : "3170000",
          "column12" : "서울특별시 금천구",
          "name" : "동해자동차정비",
          "state" : "서울특별시",
          "location" : "37.470815,126.891905",
          "water_quality" : "273",
          "sector" : "정비업소",
          "address" : "서울특별시 금천구 두산로7길 10 (독산동)",
          "phone" : "02-851-4972",
          "city" : "금천구",
          "date,provider_id" : "2019.9.10"
        }
      },
      {
        "_index" : "car_wash",
        "_type" : "_doc",
        "_id" : "pPHOcW4BGQw9X5k0OwxT",
        "_score" : 1.0,
        "_source" : {
          "privider_name" : "3170000",
          "column12" : "서울특별시 금천구",
          "name" : "SK에너지(주)이가주유소",
          "state" : "서울특별시",
          "location" : "37.472984,126.897897",
          "water_quality" : "50",
          "sector" : "주유소",
          "address" : "서울특별시 금천구 시흥대로 441 (독산동)",
          "phone" : "02-861-2241",
          "city" : "금천구",
          "date,provider_id" : "2019.9.10"
        }
      },
      {
        "_index" : "car_wash",
        "_type" : "_doc",
        "_id" : "HfHOcW4BGQw9X5k0PQ0y",
        "_score" : 1.0,
        "_source" : {
          "privider_name" : "3170000",
          "column12" : "서울특별시 금천구",
          "name" : "대하운수(주)",
          "state" : "서울특별시",
          "location" : "37.466048,126.894731",
          "water_quality" : "53",
          "sector" : "운수업 등",
          "address" : "서울특별시 금천구 범안로16길 8 (독산동)",
          "phone" : "02-805-0487",
          "city" : "금천구",
          "date,provider_id" : "2019.9.10"
        }
      }
    ]
  }
}

```





