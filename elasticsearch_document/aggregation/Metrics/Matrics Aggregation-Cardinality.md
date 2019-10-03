# Matrics Aggregation-Cardinality

cardinality : 원소의 개수라는 개념을 무한까지 확장할 수 있도록, 카디널리티를 도입

### Cardinality Aggregation

구분할수 있는 value의 갯수를 대략적으로 계산하여 나타내는 집계

```json
POST /sales/_search?size=0
{
    "aggs" : {
        "type_count" : {
            "cardinality" : {
                "field" : "type"
            }
        }
    }
}
```

결과

```json
{
    ...
    "aggregations" : {
        "type_count" : {
            "value" : 3
        }
    }
}
```

정확도 control

```json
POST /sales/_search?size=0
{
    "aggs" : {
        "type_count" : {
            "cardinality" : {
                "field" : "_doc",
                "precision_threshold": 100 
            }
        }
    }
}
```

precision_threshold 는 정확성을 위해서 메모리를 사용하는 옵션 , 고유한 갯수를 정의에 따라서 정확성이 가까워진다. 최대 값은 40,000 , 40,000 이 넘은 값을 지정해도 값은 똑같다. 기본값은 3000



갯수 는 대략적 

계한하는것은 해쉬 집합에 값을 불러오고, 그 사이즈를 반환하게된다.

많은 집합원의 갯수, 또는 많은 값들은 사용하게 되면, 높은  많은 메모리 사용량을 반환하게 된다.

그리고 노드들과 샤드집합의 관계에 있어서 많은 클러스터들의 자원들을 사용하게된다.

cardinality 집합은 hyperLogLog++ 알고리즘을 기반으로 되어있는데, value 값의 해쉬 값을 기반으로 되어있으며 아래와 같은 구성요소의 특징이 있다.

* 정확성 구성,  정확성을 위해서 얼만큼 메모리를 사용하는가에 따름 

* 낮은 cardinality 집합은 정확성이 높다.

* 메모리 사용량 고정 , 10만 에서 100만 의 고유한 값이 있다면, 메모리 사용량은 정확성에 의존하여 결정할수 있다.

  ![images / cardinality_error.png](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/cardinality_error.png)

위의 값이 사실이 아닐수도 있다. 정확성은 데이터셋에 따라 다르다.

알고리즘에 대해 자세히 알지 못하기 때문에 데이터셋에 따라서 어떻게 다른지는 파악을 하지못함 (개인의견)



Pre computed hashs

카디널리티가 높은 문자열 필드에서 필드 값의 해시를 색인에 저장 한 다음이 필드에서 카디널리티 집계를 실행하는 것이 더 빠릅니다. 이것은 클라이언트 측의 해시 값을 제공하거나 [`mapper-murmur3`](https://www.elastic.co/guide/en/elasticsearch/plugins/6.7/mapper-murmur3.html)플러그인 을 사용하여 Elasticsearch가 해시 값을 계산하도록하여 수행 할 수 있습니다.

```txt
사전 컴퓨팅 해시는 대개 CPU 및 메모리를 절약하므로 대용량 및 / 또는 상위 카디널리티 필드에서만 유용합니다. 그러나 숫자 필드에서 해싱은 매우 빠르며 원래 값을 저장하려면 해시를 저장하는 것보다 많은 메모리가 필요합니다. 낮은 카디널리티 문자열 필드에서도 마찬가지입니다. 특히 해시가 세그먼트 당 고유 값 당 최대 한 번 계산되도록하기 위해 최적화 된 경우가 그렇습니다.
```



개인적 

추가적으로 정확성을 높이기 위해서는 어떻게 변경 할 수 있을까 ??

term 쿼리+ compsite 쿼리 그 이후  버켓에 대해서  pipe Aggregation(stats_buskcets) 을 사용해서 count 를 집계하는 방법??

