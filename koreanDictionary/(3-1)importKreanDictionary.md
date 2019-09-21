#Spark를 활용해서 Elasticsearch에 데이터 삽입하기 

Excel 파일을 조작하기 위해서는 Java에서 poi를 사용하면 되지만, spark를 사용하다보니 dataframe을 사용하는 것이 더 좋을것같다라는 생각과 해보고싶다는 생각이 들었다. 그래서 spark를 활용해서 elasticsearch에 원하는 데이터를 넣어보자!! 

###Elasticsearch - Spark 연동은 아래에서 참고 하였다.

https://www.elastic.co/guide/en/elasticsearch/hadoop/6.2/spark.html

```scala
// reusing the example from Spark SQL documentation

import org.apache.spark.sql.SQLContext    
import org.apache.spark.sql.SQLContext._

import org.elasticsearch.spark.sql._      

...

// sc = existing SparkContext
val sqlContext = new SQLContext(sc)

// case class used to define the DataFrame
case class Person(name: String, surname: String, age: Int)

//  create DataFrame
val people = sc.textFile("people.txt")    
        .map(_.split(","))
        .map(p => Person(p(0), p(1), p(2).trim.toInt))
        .toDF()

people.saveToEs("spark/people")     
```



### gradle Dependency 추가

사용한 의존되는 라이브러리는 다음과 같다.

```groovy
dependencies {
  testCompile group: 'junit', name: 'junit', version: '4.12'
  implementation 'org.apache.spark:spark-core_2.11:2.4.3'
  implementation 'org.elasticsearch:elasticsearch-spark-20_2.11:6.5.3'
  implementation 'com.crealytics:spark-excel_2.11:0.12.0'
  implementation 'org.apache.spark:spark-sql_2.11:2.4.3'
}
```

crealytics 라이브러리를 사용하여 Spark-excel을 불러왔다.



###SparkSession을 통하여 excel 데이터를 dataframe 으로 불러오기

excel 파일 dataframe으로 불러오기

```scala
import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession
import org.elasticsearch.spark.sql._
object main {

  def main(args:Array[String]): Unit ={
    println("hello ES-SPARK")
    val spark = SparkSession.builder().appName("ESSpark").config("spark.master","local").getOrCreate()
    spark.conf.set("es.index.auto.create","true")
    spark.conf.set("es.nodes","127.0.0.1")
    spark.conf.set("es.port","9200")
    //load xls file
    val exceldf = spark.read
      .format("com.crealytics.spark.excel")
      .option("sheetName","sheet0")
      .option("useHeader","true")
      .load("/Users/daeyunkim/Documents/07.ELK_v6/data/korean_dic_origin/559806_60000.xls")

    exceldf.show()
    exceldf.printSchema()
    val filterData = exceldf.select("어휘","고유어 여부","품사","뜻풀이","범주")
      .withColumnRenamed("어휘","voca")
        .withColumnRenamed("고유 여부", "is_identify")
        .withColumnRenamed("품사","partOfSpeech")
        .withColumnRenamed("뜻풀이","meaning")
        .withColumnRenamed("범주","category")

    println("------")
    filterData.saveToEs("dic_sample/_doc") //("indexname/type")
  }

}

```

https://stackoverflow.com/questions/44196741/how-to-construct-dataframe-from-a-excel-xls-xlsx-file-in-scala-spark

elasticsearch에서는 index의 이름은 항상 소문자를 적어야한다. 그리고 elasticsearch 6버전 부터는 type을 하나만 지원하기 때문에 주의해야한다.



결과는 Elasticsearch에 잘 추가 된것을 알 수 있다.

```json
{
"_index": "dic_sample",
"_type": "_doc",
"_id": "I1DvcWwB7eXwunPlzEi0",
"_version": 1,
"_score": 1,
"_source": {
"voca": "곡지통",
"고유어 여부": "한자어",
"partOfSpeech": "「명사」 ",
"meaning": "목을 놓아 매우 슬프게 욺. ",
"category": "일반어 "
}
}
```





### Error(java.lang.NoSuchMethodError)

컴파일을 돌리고 나면 아래와 같이 에러가 난다 .

```scala
Caused by: java.lang.NoSuchMethodError: 
scala.Predef$.refArrayOps([Ljava/lang/Object;)Lscala/collection/mutable/ArrayOps;   
```

이유는  프로젝트에서는 2.12 버전의 scala를 사용하고 있는데, 실행되는건 2.11의 버전을 실행하고있기 때문이다. 

참고 

https://alvinalexander.com/source-code/scala-java-lang-nosuchmethoderror-compiler-message



---

###spark 2+ 부터는 

sparkSession 에 SparkContext와 SqlContext를 합쳤다고 하는데 그건 추후에 알아보자!

https://databricks.com/blog/2016/08/15/how-to-use-sparksession-in-apache-spark-2-0.html