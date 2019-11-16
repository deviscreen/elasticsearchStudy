# RealTime 화면 만들기 



이전에 만들었던 쿼리를 응용해서 만들기

```json
GET realtime_word/_search
{
  "query":{
    "range":{
      "reqdt":{
        "gte":20191019021220,
        "lte":20191019021245
      }
    }
  },
  "size":0,
  "aggs":{
    "top_5_word":{
      "terms": {"field":"searchWd"}
      
    }
  }
}
```

서버의 시간을 기준으로 웹에서 5분전 20분전 의 값을 보내면 그것에 기준하여 5분전 시간을 보내줌 

