#Spark Streaming DEMO

本例子使用 Kafka 收集数据，Spark Streaming 分析统计，并把结果存储至 Redis 中去。

---

环境

- 系统 ： centos 7
- Spark 包版本： spark-1.6.1-hadoop-2.6.0
- Scala 版本 scala 2.10.6
- IDE ： IDEA

---

1. 创建 SBT 工程

2.  引入依赖
	
	编辑 build.sbt 文件，在文件中追加如下代码
	```sbt
	libraryDependencies ++= Seq(
		"org.apache.spark" % "spark-streaming_2.10" % "1.6.1",
		"org.apache.spark" % "spark-streaming-kafka_2.10" % "1.6.1",
        "net.debasishg" %% "redisclient" % "3.0"
	)
	```

3. 代码编写
	
	- 在 src/main/scala 目录下创建包 Stream
	- 在 Stream 包下创建 Scala 类文件 Redis **并选择创建类型为 Object**

	示例代码如下
	```scala
	package Stream

	import com.redis._
	import kafka.serializer.StringDecoder
	import org.apache.spark.SparkConf
	import org.apache.spark.streaming.{Seconds, StreamingContext}
	import org.apache.spark.streaming.kafka._
	
	object Redis{
	  def main(args: Array[String]) {
	
	    if(args.length < 2){
	      System.err.println("error")
	      System.exit(1)
	    }
	
	    val Array(brokers,topics) = args
	    val kafkaParams = Map[String,String]("metadata.broker.list" -> brokers)
	    val sparkConf = new SparkConf().setAppName("Kafka")
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

4. 编辑代码

	- 菜单栏进入 File > Project Structure > Artifacts
	- 选择 「+」 > JAR > From modules with dependencies
	-  选择 Main Class 为 Stream.Redis 后一路 OK 退回代码界面
	- 菜单栏进入 Build > Build Artifacts
	- 在弹出窗口中选刚才创建的 Artifact 选择 Build 操作并等待

5. 运行代码

	编译打包好的 Jar 位于项目根目录下 out/artifacts/「your-artifact-name」下
	运行如下代码即可提交任务至 spark 中执行
	```bash
	# 运行 zookeeper 服务
	/PATH/TO/KAFKA/bin/zookeeper-server-start.sh /PATH/TO/KAFKA/config/zookeeper.properties
	# 运行 kafka 服务
	/PATH/TO/KAFKA/bin/kafka-server-start.sh /PATH/TO/KAFKA/config/server.properties
	# 运行 spark 服务
	/PATH/TO/SPARK/sbin/start-all.sh
	# 提交任务
	/PATH/TO/SPARK/bin/spark-submit  --executor-memory 1g --master spark://127.0.0.1:7077 --class Stream.Redis [path/to/artifacts.jar] [kafka-address] [topic-name]	
	```
	
6. 向 Kafka 发送数据，查看结果





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