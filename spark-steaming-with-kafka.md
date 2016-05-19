# Spark Steaming DEMO

---

本例子使用 Kafka 作为数据采集来源，Spark Steaming 进行分析统计。

---

示例代码

```scala
package Stream

import kafka.serializer.StringDecoder
import org.apache.spark.{HashPartitioner, SparkConf}
import org.apache.spark.examples.streaming.StreamingExamples
import org.apache.spark.streaming.{Minutes, Seconds, StreamingContext}
import org.apache.spark.streaming.kafka._

object Kafka {
  def main(args: Array[String]) {

    if(args.length < 2){
      System.err.println("error")
      System.exit(1)
    }

    StreamingExamples.setStreamingLogLevels()
    import org.apache.log4j.{Level,Logger}

    Logger.getLogger("spark").setLevel(Level.WARN)
    Logger.getLogger("spark.sql").setLevel(Level.WARN)
    Logger.getLogger("spark.streaming").setLevel(Level.WARN)

    val sparkConf = new SparkConf().setAppName("Kafka")
    val ssc = new StreamingContext(sparkConf, Seconds(10))

    ssc.checkpoint("checkPoint")

    val Array(brokers,topics) = args

    val topicsSet = topics.split(" ").toSet
    val kafkaParams = Map[String,String]("metadata.broker.list" -> brokers)
    val messages = KafkaUtils.createDirectStream[String,String,StringDecoder,StringDecoder](ssc,kafkaParams,topicsSet)
    val lines = messages.map(_._2)

    val words = lines.flatMap(_.split(" "))
    val wordCounts = words.map(x => (x, 1L)).reduceByKey(_ + _)

    wordCounts.print()

    ssc.start()
    ssc.awaitTermination()
  }
}

```