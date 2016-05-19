#Spark Streaming DEMO

---

本例子使用 Kafka 收集数据，Spark Streaming 分析统计，并把结果存储至 Redis 中去。

---

问题
1. 在把分析结果保存到 Redis 中时可能发生如下错误

```bash
java.io.NotSerializableException : DStream checkpointing has been enabled but the DStreams with their functions are not serializable
```

Spark 是一个分布式的系统，RDD 也是一个弹性数据集。这意味着所有的数据都有可能存在于不同的分区，但 Spark 会自动对这些数据进行 serializable 操作，使得用户使用这些数据的时候，就像使用本地数据一样。

所以此问题的根本原因就在于，当 Spark 使用用户自定义的方法时，为了使方法适用于 RDD ，Spark 会对方法进行 serializable 操作，此时，如果 Spark 发现此方法无法被 serializable 就会触发这个 Exception。

解决方法就是把要调用的自定义方法封装在一个可被 serializable  的对象中，使之能被 Spark 正确 serializable 。

其他解决方案请参考解决方案来源：[原文地址](http://stackoverflow.com/questions/22592811/task-not-serializable-java-io-notserializableexception-when-calling-function-ou)

---

示例代码：
```bash
package Stream

import com.redis._
import org.apache.spark.SparkConf
import kafka.serializer.StringDecoder
import org.apache.spark.{HashPartitioner, SparkConf}
import org.apache.spark.examples.streaming.StreamingExamples
import org.apache.spark.streaming.{Minutes, Seconds, StreamingContext}
import org.apache.spark.streaming.kafka._

object Redis{
  def main(args: Array[String]) {

//    if(args.length < 2){
//      System.err.println("error")
//      System.exit(1)
//    }

    StreamingExamples.setStreamingLogLevels()
    import org.apache.log4j.{Level,Logger}

    Logger.getLogger("spark").setLevel(Level.WARN)
    Logger.getLogger("spark.sql").setLevel(Level.WARN)
    Logger.getLogger("spark.streaming").setLevel(Level.WARN)

    //debug settings
    val param = new Array[String](2)
    param(0) = "127.0.0.1:9092"
    param(1) = "test1"
    val Array(brokers,topics) = param
    val kafkaParams = Map[String,String]("metadata.broker.list" -> brokers,"auto.offset.reset" -> "smallest")
    val sparkConf = new SparkConf().setAppName("Kafka").setMaster("local")

//    val Array(brokers,topics) = args
//    val kafkaParams = Map[String,String]("metadata.broker.list" -> brokers)
//    val sparkConf = new SparkConf().setAppName("Kafka")
    val ssc = new StreamingContext(sparkConf, Seconds(10))

    ssc.checkpoint("checkPoint")

    val topicsSet = topics.split(" ").toSet

    val messages = KafkaUtils.createDirectStream[String,String,StringDecoder,StringDecoder](ssc,kafkaParams,topicsSet)
    val lines = messages.map(_._2)

    val words = lines.flatMap(_.split(" "))
    val wordCounts = words.map(x => (x, 1L)).reduceByKey(_ + _)
    
    wordCounts.foreachRDD(rdd => {
      rdd.foreachPartition { partitionOfRecords =>
        partitionOfRecords.foreach(pair => {
          val word = pair._1.toString
          val count = pair._2.toString
          new redisClient().set(word,count)
        })
      }
    })

    wordCounts.print()

    ssc.start()
    ssc.awaitTermination()
  }
}

class redisClient extends Serializable{
  val r = new RedisClient("127.0.0.1", 6379)
  r.set("streaming", "live")

  def set(key:String, value:String){
    r.set(key,value)
  }
}
```