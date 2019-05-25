# Index Templates

Index templates 는  새로운인덱스가 만들어 질때 자동으로 적용된다.

템플릿은 오직 새로운 인덱스가 생성될때 적용되며, 템플릿을 변경하여 적용하면 기존의 인덱스들은 영향을 받지 않는다.



예제

```json
PUT _template/template_1
{
  "index_patterns": ["te*", "bar*"],
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "_doc": {
      "_source": {
        "enabled": false
      },
      "properties": {
        "host_name": {
          "type": "keyword"
        },
        "created_at": {
          "type": "date",
          "format": "EEE MMM dd HH:mm:ss Z yyyy"
        }
      }
    }
  }
}
```



template_1이 템플릿의 이름을 지정하였고, te\*, bar\* 와 같은 인덱스의 패턴을 지정할 수 있다.

alias까지 지정하여 인덱스 템플릿을 만들수 있다.

```json
PUT _template/template_1
{
    "index_patterns" : ["te*"],
    "settings" : {
        "number_of_shards" : 1
    },
    "aliases" : {
        "alias1" : {},
        "alias2" : {
            "filter" : {
                "term" : {"user" : "kimchy" }
            },
            "routing" : "kimchy"
        },
        "{index}-alias" : {} 
    }
}
```





###멀티인덱스 탬플릿

또한 여러개의 인덱스 템플릿을 적용할때는 order 라는 파라미터를 추가하여 통합되는 순서를 적용할 수 있다.

이 순서에 따라 작은 숫자부터 적용되고, 큰숫자는 나중에 통합된다.

```json
PUT /_template/template_1
{
    "index_patterns" : ["*"],
    "order" : 0,
    "settings" : {
        "number_of_shards" : 1
    },
    "mappings" : {
        "_doc" : {
            "_source" : { "enabled" : false }
        }
    }
}

PUT /_template/template_2
{
    "index_patterns" : ["te*"],
    "order" : 1,
    "settings" : {
        "number_of_shards" : 1
    },
    "mappings" : {
        "_doc" : {
            "_source" : { "enabled" : true }
        }
    }
}
```



###템플릿 버전 관리

선택적으로 version 숫자를 추가 할 수 있다.

이는 별다른 동작을 하지 않는 선택 옵션이며, 외부관리에서 사용가능하도록 제공하는 것

```json
PUT /_template/template_1
{
    "index_patterns" : ["*"],
    "order" : 0,
    "settings" : {
        "number_of_shards" : 1
    },
    "version": 123
}
```



템플릿의 버전은 다음과 같이 조회가 가능하다.

```json
GET /_template/template_1?filter_path=*.version
```

결과 : 

```json
{
    "template_1" : {
        "version" : 123
    }
}
```

