# Aggregation

elasticsearch 지원 집계



* Bucket 집계
* Metric집계
* Metrix 집계
* Pipeline 집계



### Bucket 집계

컨텍스트에 있는 각 도큐먼트가 어떤 버킷에 속해 있는지 결정해 평가

bucket 집계는 개별 버킷과 버킷에 속하는 도큐먼트가 있는 버킷 집합을 갖는다.

sql 에서는 group by 절이 있는것을 생각하면됨 

```sql
select column , count(*) from table1 group by column1;
```

column1에 속한 값을 나눈후 column 개별값의 도큐먼트 수를 반환

bucket 집계쿼리 에서 최상단 혹은 가장 바깥 레벨에 위치 

다른 bucket집계 내부에 중첩해 사용할 수 있다.



어떤 부분집합에 관련이 있는지 분석

bucket 집계의 각 타입은 여러 segment 혹은 버킷으로 데이터를 분할 

* 문자열 데이터 버킷팅 (terms 집계)
  * 가장 인기있는 카테고리 , 사용자가 가장 많이 검색한 카테고리 ...
* 숫자 데이터 버킷팅
  * Histogram : 숫자 범위, 간격을 가지고 쿼리를 사용하여 버킷을 분할 
  * Range : from,to 를 사용하여 집계에 해당하는 것을 버킷으로 분할 
* 필터 데이터 집계
  * aggs 가 아닌 query 문 에서 term 필터를 추가하고 집계
* 중첩 집계
  * query 로 집계텍스트를 제한하고 그 이후, 버킷하나를 만든후, 다시 metric 집계를 진행
* 맞춤형 조건 버킷팅
* 날자 또는 시간 데이터 버킷팅
* 지리 공간 데이터 버킷팅



### Metric 집계

**필드에서 숫자 타입**으로 동작, 주어진 컨텍스트에서 숫자 필드의 집계값을 계산하는데 사용

```sql
select avg(score) from results;
```

여기서 말하는것은 **전체 테이블 즉 모든 값**들을 말한다.

bucket에 중첩되어 사용할 수 있다.

* 합계, 평균, 최소, 최대 집계( sum, avg, min, max)

* 통계 및 확장 통계 집계 ( stats, extended_stats)

* 카디널리티 집계 (cardinality)

  

### Matrix 집계

여러 필드에서 동작하고 쿼리 컨텍스트 내의 모든 도큐먼트에 걸쳐 메트릭을 계산

Matrix 집계는 bucket집계에 중첩해 사용할 수 있지만, Bucket 집계는 Matrix 내부에 중첩될수 없다.



###Pipiline 집계

다른 타입의 집계결과를 다시 집계할 수 있는 상위 레벨의 집계

파생상품과 같은 예제를 계산할 때 유용하게 사용이 가능 

