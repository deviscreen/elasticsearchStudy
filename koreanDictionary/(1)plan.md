# 사전 만들기 프로젝트(1) - 데이터 다운로드 및 template 만들기

국어사전검색을 구현하는 프로젝트를 만들어보자!!



sample Dataset

```json
{
  "voca":"가위",//[0]
  "isNative":"한자어",//[2]
  "partOfSpeech": "명사",//[10]
  "meaning":"옷감, 종이, 머리털 따위를 자르는 기구...", //[15]
  "category":"일반어", //[17]3
}
```

출처 : 표준 국어 대사전



```json
PUT _template/dic_template
{
  "index_patterns": ["dic*"],
  "settings": {
    "index": {
      "number_of_shards": 5,
      "number_of_replicas": 0
    },
    "analysis": {
      "analyzer": {
        "ngram_analyzer": {
          "type": "custom",
          "tokenizer": "ngram_tokenizer",
          "filter": [
            "lowercase",
            "trim"
          ]
        },
        "edge_ngram_analyzer": {
          "type": "custom",
          "tokenizer": "edge_ngram_tokenizer",
          "filter": [
            "lowercase",
            "trim",
            "edge_ngram_filter_front"
          ]
        },
        "edge_ngram_analyzer_back": {
          "type": "custom",
          "tokenizer": "edge_ngram_tokenizer",
          "filter": [
            "lowercase",
            "trim",
            "edge_ngram_filter_back"
          ]
        }
      },
      "tokenizer": {
        "ngram_tokenizer": {
          "type": "nGram",
          "min_gram": "1",
          "max_gram": "50",
          "token_chars": [
            "letter",
            "digit",
            "punctuation",
            "symbol"
          ]
        },
        "edge_ngram_tokenizer": {
          "type": "edgeNGram",
          "min_gram": "1",
          "max_gram": "50",
          "token_chars": [
            "letter",
            "digit",
            "punctuation",
            "symbol"
          ]
        }
      },
      "filter": {
        "edge_ngram_filter_front": {
          "type": "edgeNGram",
          "min_gram": "1",
          "max_gram": "50",
          "side": "front"
        },
        "edge_ngram_filter_back": {
          "type": "edgeNGram",
          "min_gram": "1",
          "max_gram": "50",
          "side": "back"
        }
      }
    }
  },
    "mappings": {
      "_doc": {
        "properties": {
          "voca": {
            "type": "text",
            "analyzer": "ngram_analyzer",
            "search_analyzer": "ngram_analyzer",
            "boost": 3
          },
          "isNative": {
            "type": "keyword"
          },
          "partOfSpeech": {
            "type": "keyword"
          },
          "meaning": {
            "type": "text",
            "analyzer": "edge_ngram_analyzer",
            "search_analyzer": "edge_ngram_analyzer"
          },
          "category": {
            "type": "keyword"
          }
        }
      }
    }
}

```

인덱스 생성 및 예제 데이터 삽입

```json
PUT dic_korean

POST dic_korean/_doc/1
{
  "voca":"가위",
  "isNative":"한자어",
  "partOfSpeech": "명사",
  "meaning":"옷감, 종이, 머리털 따위를 자르는 기구...", 
  "category":"일반어"
}

GET dic_korean/_mapping

```

데이터 검색 확인

```json
GET dic_korean/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "voca": "가위"
          }
        }
      ]
    }
  }
}

GET dic_korean/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "meaning": "자르"
          }
        }
      ]
    }
  }
}
```

