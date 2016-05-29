# 从日志统计应用安装次数

日志各字段之间以空格符分割，格式如下

```
用户ID 应用ID 安装时间
```

示例代码如下

```bash
package Stream

import com.redis._
import kafka.serializer.StringDecoder
import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}
import org.apache.spark.streaming.kafka._

object AppCount {
  def main(args: Array[String]) {

    // set log level
    import org.apache.log4j.{Level, Logger}

    Logger.getLogger("org.apache.spark").setLevel(Level.WARN)
    Logger.getLogger("org.apache.spark.sql").setLevel(Level.WARN)
    Logger.getLogger("org.apache.spark.streaming").setLevel(Level.WARN)

    // set topic and
    val Array(brokers, topics) = Array("localhost:9092", "appInstall")
    val kafkaParams = Map[String, String]("metadata.broker.list" -> brokers)
    val sparkConf = new SparkConf().setAppName("Kafka").setMaster("local")
    val ssc = new StreamingContext(sparkConf, Seconds(10))

    ssc.checkpoint("checkPoint")

    val topicsSet = topics.split(" ").toSet

    val messages = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder](ssc, kafkaParams, topicsSet)
    val lines = messages.map(_._2)

    val pids = lines.map(x => getPID(x))

    pids.print()

    val InstallCounts = pids.map(x => (x, 1L)).reduceByKey(_ + _)

    InstallCounts.foreachRDD(rdd => {
      rdd.foreachPartition { partitionOfRecords =>
        partitionOfRecords.foreach(pair => {
          val word = pair._1.toString
          val count = pair._2.toString
          new redisClient().hset("appInstallCount", word, count)
        })
      }
    })

    InstallCounts.print()

    ssc.start()
    ssc.awaitTermination()
  }

  def getPID(log: String): String = {
    val x = log.split(" ")
    val pid = x(1)
    pid
  }
}

class redisClient extends Serializable {
  val r = new RedisClient("127.0.0.1", 6379)
  r.set("streaming", "live")

  def set(key: String, value: String) {
    r.set(key, value)
  }

  def hset(key: String, field: String, value: String) {
    r.hset(key, field, value);
  }
}
```