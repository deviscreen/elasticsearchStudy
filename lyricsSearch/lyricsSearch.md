#Nori 사용 해서 노래 가사 검색하기

특정 가사를 검색하면 노래의 타이틀을 검색하는 프로그램 짜기 

예제 데이터는 다음과 같다.
```json
{
     "title" : "사랑에 연습이 있었다면",
     "singer ":"임재현",
     "lyrics":"사랑에 연습이 있었다면
                우리는 달라졌을까
                내가 널 만난 시간 혹은 그 장소
                상황이 달랐었다면
                우린 맺어졌을까
                하필 넌
                왜 내가 그렇게 철없던 시절에
                나타나서 그렇게 예뻤니
                너처럼 좋은 여자가
                왜 날 만나서 그런
                과분한 사랑 내게 줬는지
                우리 다시 그때로 돌아가자는게
                그게 미친말인가
                정신나간 소린가
                나는 더 잘 할 수 있고
                다신 울리지 않을
                자신있는데 그게 왜 말이 안돼
                시간이 너무 흘러 알게되었는데
                너를 울리지 않고 아껴주는법
                세월은 왜 철없는 날
                기다려주지 않고
                흘러갔는지 야속해
                지금 너 만나는 그에게도
                내게 그랬던 것처럼
                예쁘게 웃어주니
                너처럼 좋은 여자의
                사랑 받는 그 남자 너무 부러워
                넌 행복하니
                니옆에 지금 그 남자가 있는게
                우리 다시 맺어질수가 없는 이윤가
                나는 더 잘 할 수 있고
                다신 울리지 않을
                자신있는데 그게 왜 말이 안돼
                시간이 너무 흘러 알게 되었는데
                너를 울리지 않고 아껴주는 법
                세월은 왜 널 잊는 법을
                알려주지 않고 흘러갔는지"

}
```

순서 : 
1. elasticsearch analyzer nori 사용 맵핑 설정
2. 데이터 삽입하기 
3. 가사 검색해보기 

### 1. Elasticsearch Analyzer Nori 적용하기 

가사를 검색할것이기 때문에 text 데이터타입을 사용하자!! 
text 데이터 타입은 전문 검색이 가능하다는 점이 특징 
text 타입으로 데이터를 색인하면 전체 텍스트가 토큰화되어 새엉되며 특정 단어를 검색하는 것이 가능해진다.

```json
PUT lyricsSearch
{
    "settings":{
        "index":{
            "analysis":{
                "tokenizer":{
                    "nori":{
                        "type":"nori_tokenizer",
                        "decompound_mode":"mixed",
                        "user_dictionary":"userdict_ko.txt"
                    }
                },
                "analyzer":{
                    "my_analyzer":{
                        "type":"custom",
                        "tokenizer":"nori"
                    }
                }
            }
        }
    },
    "mappings":{
        "_doc":{
            "properties":{
                "lyrics":{
                    "type":"text"
                },
                "singer":{
                    "type":"keyword"
                },
                "title":{
                    "type":"text"
                }
            }
        }
    }
}
```

검색하는 쿼리는 다음과 같다.
```json
GET lyrics_search/_search
{
  "query":{
    "match":{
      "lyrics": "사랑"
    }
  }
}
```
제대로 나오는 것 같다.

이렇게 긴 텍스트는 어떻게 인덱싱 되는지 알아보기 위해 확인 해보았다.

```json
POST lyrics_search/_analyze
{
  "analyzer":"my_analyzer",
  "text": "사랑에 연습이 있었다면 우리는 달라졌을까 내가 널 만난 시간 혹은 그 장소 상황이 달랐었다면 우린 ..."
}
```

결과는 아래와 같이 토큰이 나누어 있었다.
```json
  {"tokens" : [
    {
      "token" : "사랑",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "에",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "연습",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "이",
      "start_offset" : 6,
      "end_offset" : 7,
      "type" : "word",
      "position" : 3
    },
    ...
    {
      "token" : "달라졌",
      "start_offset" : 17,
      "end_offset" : 20,
      "type" : "word",
      "position" : 9,
      "positionLength" : 2
    },
    {
      "token" : "달라지",
      "start_offset" : 17,
      "end_offset" : 20,
      "type" : "word",
      "position" : 9
    },
    {
      "token" : "었",
      "start_offset" : 17,
      "end_offset" : 20,
      "type" : "word",
      "position" : 10
    }
    ...]}
    }
```

위와 같은 이유는 tokenizer nori 모드를 mixed 모드를 사용했기 때문에 `사랑에` 라는 단어는 `사랑, 에`  두 가지로 나타난다.



nori tokenizer에서 decompound_mode를 discard 로 적용하였을때

```json
{
  "tokens" : [
    {
      "token" : "사랑",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "에",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "연습",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "이",
      "start_offset" : 6,
      "end_offset" : 7,
      "type" : "word",
      "position" : 3
    }
    ...
    {
      "token" : "달라지",
      "start_offset" : 17,
      "end_offset" : 20,
      "type" : "word",
      "position" : 9
    },
    {
      "token" : "었",
      "start_offset" : 17,
      "end_offset" : 20,
      "type" : "word",
      "position" : 10
    },
```



nori에서  tokenizer 에서 decompound_mode 모드를 none으로 입력하였을때

```json
{
  "tokens" : [
    {
      "token" : "사랑",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "에",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "연습",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "이",
      "start_offset" : 6,
      "end_offset" : 7,
      "type" : "word",
      "position" : 3
    },
    {
      "token" : "있",
      "start_offset" : 8,
      "end_offset" : 9,
      "type" : "word",
      "position" : 4
    },
    {
      "token" : "었",
      "start_offset" : 9,
      "end_offset" : 10,
      "type" : "word",
      "position" : 5
    },
    {
      "token" : "다면",
      "start_offset" : 10,
      "end_offset" : 12,
      "type" : "word",
      "position" : 6
    },
  ....
    {
      "token" : "달라졌",
      "start_offset" : 17,
      "end_offset" : 20,
      "type" : "word",
      "position" : 9
    },
    {
      "token" : "을까",
      "start_offset" : 20,
      "end_offset" : 22,
      "type" : "word",
      "position" : 10
    },
...
```

각 모드마다 다르기 때문에 match 쿼리를 사용해서 더 어느정도까지 검색을 할것인지에 대해 잘 생각하고 해야할것 같다.

-----

연습 쿼리 

```json
PUT lyrics_search
{
    "settings":{
        "index":{
            "analysis":{
                "tokenizer":{
                    "nori":{
                        "type":"nori_tokenizer",
                        "decompound_mode":"mixed",
                        "user_dictionary":"userdict_ko.txt"
                    }
                },
                "analyzer":{
                    "my_analyzer":{
                        "type":"custom",
                        "tokenizer":"nori"
                    }
                }
            }
        }
    },
    "mappings":{
        "_doc":{
            "properties":{
                "lyrics":{
                    "type":"text"
                },
                "singer":{
                    "type":"keyword"
                },
                "title":{
                    "type":"text"
                }
            }
        }
    }
}

POST lyrics_search/_doc/1
{
     "title" : "사랑에 연습이 있었다면",
     "singer ":"임재현",
     "lyrics": "사랑에 연습이 있었다면 우리는 달라졌을까 내가 널 만난 시간 혹은 그 장소 상황이 달랐었다면 우린 맺어졌을까 하필 넌 왜 내가 그렇게 철없던 시절에 나타나서 그렇게 예뻤니  너처럼 좋은 여자가  왜 날 만나서 그런  과분한 사랑 내게 줬는지  우리 다시 그때로 돌아가자는게  그게 미친말인가  정신나간 소린가  나는 더 잘 할 수 있고  다신 울리지 않을  자신있는데 그게 왜 말이 안돼  시간이 너무 흘러 알게되었는데  너를 울리지 않고 아껴주는법 세월은 왜 철없는 날  기다려주지 않고 흘러갔는지 야속해  지금 너 만나는 그에게도  내게 그랬던 것처럼 예쁘게 웃어주니  너처럼 좋은 여자의 사랑 받는 그 남자 너무 부러워 넌 행복하니 니옆에 지금 그 남자가 있는게  우리 다시 맺어질수가 없는 이윤가  나는 더 잘 할 수 있고  다신 울리지 않을 자신있는데 그게 왜 말이 안돼 시간이 너무 흘러 알게 되었는데 너를 울리지 않고 아껴주는 법 세월은 왜 널 잊는 법을 알려주지 않고 흘러갔는지"

}
GET lyrics_search/_search
{
  "query":{
    "match":{
      "lyrics": "사랑에 있었다면"
    }
  }
}

GET lyrics_search/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "lyrics": "사랑에"
          }
        },
        {
          "term":{
            "lyrics": "있었다면"
          }
          
        }
      ]
    }
  }
}

POST lyrics_search/_analyze
{
  "analyzer":"my_analyzer",
  "text": "사랑에 연습이 있었다면 우리는 달라졌을까 내가 널 만난 시간 혹은 그 장소 상황이 달랐었다면 우린 맺어졌을까 하필 넌 왜 내가 그렇게 철없던 시절에 나타나서 그렇게 예뻤니  너처럼 좋은 여자가  왜 날 만나서 그런  과분한 사랑 내게 줬는지  우리 다시 그때로 돌아가자는게  그게 미친말인가  정신나간 소린가  나는 더 잘 할 수 있고  다신 울리지 않을  자신있는데 그게 왜 말이 안돼  시간이 너무 흘러 알게되었는데  너를 울리지 않고 아껴주는법 세월은 왜 철없는 날  기다려주지 않고 흘러갔는지 야속해  지금 너 만나는 그에게도  내게 그랬던 것처럼 예쁘게 웃어주니  너처럼 좋은 여자의 사랑 받는 그 남자 너무 부러워 넌 행복하니 니옆에 지금 그 남자가 있는게  우리 다시 맺어질수가 없는 이윤가  나는 더 잘 할 수 있고  다신 울리지 않을 자신있는데 그게 왜 말이 안돼 시간이 너무 흘러 알게 되었는데 너를 울리지 않고 아껴주는 법 세월은 왜 널 잊는 법을 알려주지 않고 흘러갔는지"
}

## decompound Mode : Discard
PUT lyrics_discard
{
    "settings":{
        "index":{
            "analysis":{
                "tokenizer":{
                    "nori":{
                        "type":"nori_tokenizer",
                        "decompound_mode":"discard",
                        "user_dictionary":"userdict_ko.txt"
                    }
                },
                "analyzer":{
                    "my_analyzer":{
                        "type":"custom",
                        "tokenizer":"nori"
                    }
                }
            }
        }
    },
    "mappings":{
        "_doc":{
            "properties":{
                "lyrics":{
                    "type":"text"
                },
                "singer":{
                    "type":"keyword"
                },
                "title":{
                    "type":"text"
                }
            }
        }
    }
}

##decompoundMode : discard Test
POST lyrics_discard/_analyze
{
  "analyzer":"my_analyzer",
  "text": "사랑에 연습이 있었다면 우리는 달라졌을까 내가 널 만난 시간 혹은 그 장소 상황이 달랐었다면 우린 맺어졌을까 하필 넌 왜 내가 그렇게 철없던 시절에 나타나서 그렇게 예뻤니  너처럼 좋은 여자가  왜 날 만나서 그런  과분한 사랑 내게 줬는지  우리 다시 그때로 돌아가자는게  그게 미친말인가  정신나간 소린가  나는 더 잘 할 수 있고  다신 울리지 않을  자신있는데 그게 왜 말이 안돼  시간이 너무 흘러 알게되었는데  너를 울리지 않고 아껴주는법 세월은 왜 철없는 날  기다려주지 않고 흘러갔는지 야속해  지금 너 만나는 그에게도  내게 그랬던 것처럼 예쁘게 웃어주니  너처럼 좋은 여자의 사랑 받는 그 남자 너무 부러워 넌 행복하니 니옆에 지금 그 남자가 있는게  우리 다시 맺어질수가 없는 이윤가  나는 더 잘 할 수 있고  다신 울리지 않을 자신있는데 그게 왜 말이 안돼 시간이 너무 흘러 알게 되었는데 너를 울리지 않고 아껴주는 법 세월은 왜 널 잊는 법을 알려주지 않고 흘러갔는지"
}

##none
PUT lyrics_none
{
    "settings":{
        "index":{
            "analysis":{
                "tokenizer":{
                    "nori":{
                        "type":"nori_tokenizer",
                        "decompound_mode":"none",
                        "user_dictionary":"userdict_ko.txt"
                    }
                },
                "analyzer":{
                    "my_analyzer":{
                        "type":"custom",
                        "tokenizer":"nori"
                    }
                }
            }
        }
    },
    "mappings":{
        "_doc":{
            "properties":{
                "lyrics":{
                    "type":"text"
                },
                "singer":{
                    "type":"keyword"
                },
                "title":{
                    "type":"text"
                }
            }
        }
    }
}

POST lyrics_none/_analyze
{
  "analyzer":"my_analyzer",
  "text": "사랑에 연습이 있었다면 우리는 달라졌을까 내가 널 만난 시간 혹은 그 장소 상황이 달랐었다면 우린 맺어졌을까 하필 넌 왜 내가 그렇게 철없던 시절에 나타나서 그렇게 예뻤니  너처럼 좋은 여자가  왜 날 만나서 그런  과분한 사랑 내게 줬는지  우리 다시 그때로 돌아가자는게  그게 미친말인가  정신나간 소린가  나는 더 잘 할 수 있고  다신 울리지 않을  자신있는데 그게 왜 말이 안돼  시간이 너무 흘러 알게되었는데  너를 울리지 않고 아껴주는법 세월은 왜 철없는 날  기다려주지 않고 흘러갔는지 야속해  지금 너 만나는 그에게도  내게 그랬던 것처럼 예쁘게 웃어주니  너처럼 좋은 여자의 사랑 받는 그 남자 너무 부러워 넌 행복하니 니옆에 지금 그 남자가 있는게  우리 다시 맺어질수가 없는 이윤가  나는 더 잘 할 수 있고  다신 울리지 않을 자신있는데 그게 왜 말이 안돼 시간이 너무 흘러 알게 되었는데 너를 울리지 않고 아껴주는 법 세월은 왜 널 잊는 법을 알려주지 않고 흘러갔는지"
}

PUT lyrics_test
{
    "settings":{
        "index":{
            "analysis":{
                "tokenizer":{
                    "nori":{
                        "type":"nori_tokenizer",
                        "decompound_mode":"none",
                        "user_dictionary":"userdict_ko.txt"
                    }
                },
                "analyzer":{
                    "my_analyzer":{
                        "tokenizer":"nori"
                    }
                }
            }
        }
    },
    "mappings":{
        "_doc":{
            "properties":{
                "lyrics":{
                    "type":"text"
                },
                "singer":{
                    "type":"keyword"
                },
                "title":{
                    "type":"text"
                }
            }
        }
    }
}
GET lyrics_test/_settings

GET lyrics_search/_stats/segments

GET twitter_ko/_stats/segments
GET twitter_ko/_search
```

