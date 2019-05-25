# Indices Stats

인덱스 레벨의 stat의 통계치를 제공한다.



모든인덱스의 정보의 통계가 출력된다

```json
GET /_stats

특정 인덱스도 가능
GET /index1, index2/_stats
```



| 속성            | 내용                                                         |
| --------------- | ------------------------------------------------------------ |
| `docs`          | 문서 또는 삭제된 문서(아직 병합되지 않은 문서)의 수. 인덱스 갱신에 영향을 받음. |
| `store`         | 인덱스 사이즈                                                |
| `indexing`      | 색인 통계, 문서 단위의 통계를 제공하기 위해 쉼표(,)로 구분된 `types` 목록과 결합될 수 있습니다. |
| `get`           | 조회 통계(조회 실패 포함)                                    |
| `search`        | 검색 통계(제안 통계 포함). 추가적으로 `groups`라는 파라미터를 사용하여 통계 결과를 그룹핑(검색 동작은 1개 이상의 그룹에 속할 수 있음) 할 수 있습니다. `groups` 파라미터는 쉼표(,) 방식의 목록을 지원합니다. 전체의 경우에는 `_all`을 사용하시면 됩니다. |
| `segments`      | 열린 세그먼트의 메모리 사용량을 확인할 수 있습니다. `include_segment_file_sizes`설정을 사용하면 루씬 인덱스 파일 각각의 디스크 사용량까지 확인할 수 있습니다. |
| `completion`    | 완성 제안 통계                                               |
| `fielddata`     | 필드 데이터 통계                                             |
| `flush`         | 정리 통계                                                    |
| `merge`         | 병합 통계                                                    |
| `request_cache` | [파편 요청 캐시](https://elasticsearch.oofbird.net/shard-request-cache.md) 통계 |
| `refresh`       | 갱신 통계                                                    |
| `warmer`        | Warmer 통계 (더 이상 사용은 안됨)                            |
| `translog`      | 트랜잭션로그 통계                                            |



```json
GET /twitter/_stats/merge,segments
```

다음과 같이 특정 인덱스의 특정 정보를 가지고 올수도있다.



샤드가 클러스터간이동할때는 다른노드에서 생성되었던 통계정보는초기화 되나, 반대로 생각하면, 샤드를 남겨놓는다고볼수 있다.



# Indices Segments

낮은 레벨의 segments 정보들을 제공



```json
GET /twitter/_segments
```

각 샤드에 segments 들에 대한 정보를 출력한다.

```json
"indices" : {
    "twitter" : {
      "shards" : {
        "0" : [
          {
            "routing" : {
              "state" : "STARTED",
              "primary" : true,
              "node" : "uCyuJNg-TVu3E7g7nzRAZw"
            },
            "num_committed_segments" : 0,
            "num_search_segments" : 0,
            "segments" : { }
          }
        ],
        "1" : [
          {
            "routing" : {
              "state" : "STARTED",
              "primary" : true,
              "node" : "uCyuJNg-TVu3E7g7nzRAZw"
            },
            "num_committed_segments" : 0,
            "num_search_segments" : 0,
            "segments" : { }
          }
        ],
        "2" : [
          {
            "routing" : {
              "state" : "STARTED",
              "primary" : true,
              "node" : "uCyuJNg-TVu3E7g7nzRAZw"
            },
            "num_committed_segments" : 1,
            "num_search_segments" : 1,
            "segments" : {
              "_0" : {
                "generation" : 0,
                "num_docs" : 1,
                "deleted_docs" : 0,
                "size_in_bytes" : 4495,
                "memory_in_bytes" : 1701,
                "committed" : true,
                "search" : true,
                "version" : "7.5.0",
                "compound" : true,
                "attributes" : {
                  "Lucene50StoredFieldsFormat.mode" : "BEST_SPEED"
                }
              }
            }
          }
        ],
        "3" : [
          {
            "routing" : {
              "state" : "STARTED",
              "primary" : true,
              "node" : "uCyuJNg-TVu3E7g7nzRAZw"
            },
            "num_committed_segments" : 0,
            "num_search_segments" : 0,
            "segments" : { }
          }
        ],
        "4" : [
          {
            "routing" : {
              "state" : "STARTED",
              "primary" : true,
              "node" : "uCyuJNg-TVu3E7g7nzRAZw"
            },
            "num_committed_segments" : 0,
            "num_search_segments" : 0,
            "segments" : { }
          }
        ]
      }
    }
  }
```

#### _0

세그먼트에 대한 이름이 JSON 객체의 키로 구성됩니다. 이 이름은 파일을 생성할 때 사용되며, 이 세그먼트가 포함된 파편의 디렉터리 안에 이 이름으로 시작하여 파일을 만듭니다.

#### generation

새로운 세그먼트 기록이 필요할 경우 증가하는 generation 번호 입니다. 세그먼트 이름은 이 숫자로부터 파생됩니다.

#### num_docs

세그먼트 안에 삭제되지 않고 저장된 문서의 수 입니다.

#### deleted_docs

세그먼트 안에 저장되어있는 삭제된 문서의 수 입니다. 만약 이 숫자가 0보다 크다면, 병합처리를 통해서 공간을 재활용할 수 있다는 것 입니다.

#### size_in_bytes

바이트 단위로, 세그먼트가 사용한 디스크 공간입니다.

#### memory_in_bytes

세그먼트는 검색을 좀 더 효율적으로 처리하기 위하여 일부 데이터를 메모리에 저장해야합니다. 이 숫자는 이러한 목적으로 사용된 메모리 공간의 크기를 가리킵니다. 만약 -1이라면 Elasticsearch에서 이 숫자를 계산할 수 없는 상태라고 보시면 됩니다.

#### committed

세그먼트가 파일 시스템에 동기화가 되었는지 여부 입니다. 커밋된 세그먼트는 재기동 되어도 문제 없습니다. 만약 커밋에 실패한다고 해도 걱정 안하셔도 됩니다. 커밋되지 않은 세그먼트 정보는 트랜젝션로그에 저장이 되며 다음 기동시 Elasticsearch가 변경사항을 다시 적용하여 세그먼트를 복구해줍니다.

#### search

검색이 가능한지 여부 입니다. 만약 이 항목이 *false*라면 아마도 세그먼트가 디스크에 기록은 되었지만 검색이 가능하도록 갱신이 아직 안되어 있는 상태라고 보시면 됩니다.

#### version

이 세그먼트에 사용된 루씬의 버전입니다.

#### compound

세그먼트가 통합파일에 저장된지 여부 입니다. 만약 *true*라면 루씬에서 파일 디스크립터 공간을 절약하기 위하여 다수의 세그먼트 파일을 한개로 병합했다는 의미입니다.





Verbose (말이 많은) 이라는 것을 사용하게 되면 추가적인 정보를 준다고하는데 

```json
GET /twitter/_segments?verbose=true
```

아래를 보면 segments 의 갯수가 유효한 샤드에 대해 자세한 정보를 주는데 무슨말인지 모르겠다.

```json
 "2" : [
          {
            "routing" : {
              "state" : "STARTED",
              "primary" : true,
              "node" : "uCyuJNg-TVu3E7g7nzRAZw"
            },
            "num_committed_segments" : 1,
            "num_search_segments" : 1,
            "segments" : {
              "_0" : {
                "generation" : 0,
                "num_docs" : 1,
                "deleted_docs" : 0,
                "size_in_bytes" : 4495,
                "memory_in_bytes" : 1701,
                "committed" : true,
                "search" : true,
                "version" : "7.5.0",
                "compound" : true,
ram_tree" : [
                  {
                    "description" : "postings [PerFieldPostings(segment=_0 formats=1)]",
                    "size_in_bytes" : 1189,
                    "children" : [
                      {
                        "description" : "format 'Lucene50_0' [BlockTreeTermsReader(fields=5,delegate=Lucene50PostingsReader(positions=true,payloads=false))]",
                        "size_in_bytes" : 1117,
                        "children" : [
                          {
                            "description" : "field '_id' [BlockTreeTerms(seg=_0 terms=1,postings=1,positions=-1,docs=1)]",
                            "size_in_bytes" : 217,
                            "children" : [
                              {
                                "description" : "term index [FST(input=BYTE1,output=ByteSequenceOutputs]",
                                "size_in_bytes" : 57
                              }
                            ]
                          },
                          {
                            "description" : "field 'message' [BlockTreeTerms(seg=_0 terms=3,postings=3,positions=3,docs=1)]",
                            "size_in_bytes" : 217,
                            "children" : [
                              {
                                "description" : "term index [FST(input=BYTE1,output=ByteSequenceOutputs]",
                                "size_in_bytes" : 57
                              }
                            ]
                          },
                          {
                            "description" : "field 'message.keyword' [BlockTreeTerms(seg=_0 terms=1,postings=1,positions=-1,docs=1)]",
                            "size_in_bytes" : 217,
                            "children" : [
                              {
                                "description" : "term index [FST(input=BYTE1,output=ByteSequenceOutputs]",
                                "size_in_bytes" : 57
                              }
                            ]
                          },
                          {
                            "description" : "field 'user' [BlockTreeTerms(seg=_0 terms=1,postings=1,positions=1,docs=1)]",
                            "size_in_bytes" : 217,
                            "children" : [
                              {
                                "description" : "term index [FST(input=BYTE1,output=ByteSequenceOutputs]",
                                "size_in_bytes" : 57
                              }
                            ]
                          },
                          {
                            "description" : "field 'user.keyword' [BlockTreeTerms(seg=_0 terms=1,postings=1,positions=-1,docs=1)]",
                            "size_in_bytes" : 217,
                            "children" : [
                              {
                                "description" : "term index [FST(input=BYTE1,output=ByteSequenceOutputs]",
                                "size_in_bytes" : 57
                              }
                            ]
                          },
                          {
                            "description" : "delegate [Lucene50PostingsReader(positions=true,payloads=false)]",
                            "size_in_bytes" : 32
                          }
                        ]
                      }
                    ]
                  },
                  {
                    "description" : "norms [Lucene70NormsProducer(fields=2)]",
                    "size_in_bytes" : 128
                  },
                  {
                    "description" : "docvalues [PerFieldDocValues(formats=1)]",
                    "size_in_bytes" : 68,
                    "children" : [
                      {
                        "description" : "format 'Lucene70_0' [org.apache.lucene.codecs.lucene70.Lucene70DocValuesProducer@d35dfa9]",
                        "size_in_bytes" : 48
                      }
                    ]
                  },
                  {
                    "description" : "stored fields [CompressingStoredFieldsReader(mode=FAST,chunksize=16384)]",
                    "size_in_bytes" : 312,
                    "children" : [
                      {
                        "description" : "stored field index [CompressingStoredFieldsIndexReader(blocks=1)]",
                        "size_in_bytes" : 312,
                        "children" : [
                          {
                            "description" : "doc base deltas",
                            "size_in_bytes" : 88
                          },
                          {
                            "description" : "start pointer deltas",
                            "size_in_bytes" : 88
                          }
                        ]
                      }
                    ]
                  },
                  {
                    "description" : "points [org.apache.lucene.codecs.lucene60.Lucene60PointsReader@2eb87702]",
                    "size_in_bytes" : 4,
                    "children" : [
                      {
                        "description" : "_seq_no [org.apache.lucene.util.bkd.BKDReader@74a2bde]",
                        "size_in_bytes" : 1
                      },
                      {
                        "description" : "post_date [org.apache.lucene.util.bkd.BKDReader@17404f60]",
                        "size_in_bytes" : 1
                      },
                      {
                        "description" : "score [org.apache.lucene.util.bkd.BKDReader@342b02b6]",
                        "size_in_bytes" : 2
                      }
                    ]
                  }
                ],
                "attributes" : {
                  "Lucene50StoredFieldsFormat.mode" : "BEST_SPEED"
                }
              }
            }
          }
        ],
```

