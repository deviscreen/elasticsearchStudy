# InjestAPI

https://www.elastic.co/guide/en/elasticsearch/reference//6.2/ingest-apis.html

```json

PUT _ingest/pipeline/fullname
{
  "description" : "describe pipeline",
  "processors" : [
    {
      "set" : {
        "field": "fullname",
        "value": ["{{lastname}}","{{firstname}}"]
      }
    }
  ]
}

PUT injestdb/_doc/1?pipeline=fullname
{
  "lastname": "Kim",
  "firstname": "Morris"
}
GET injestdb/_search
{
  "_source":["fullname"],
  "query":{
    "match_all":{
      
    }
  }
}
GET injestdb/_search
{
  "query":{
    "match_all":{
      
    }
  }
}

DELETE injestdb/_doc/1
###
PUT testdb

PUT _ingest/pipeline/calculate
{
  "description" : "describe pipeline",
  "processors" : [
    {
      "set" : {
        "field": "fullname",
        "value": "{{num1}}-{{num2}}"
      }
    }
  ]
}

POST testdb/_doc/1?pipeline=calculate
{
  "num1":5,
  "num2":6
}
GET testdb/_search
```

