# Spark Streaming DEMO

本例子使用 NetCat 工具发送数据，并用 Spark Steaming 进行分析统计。

##环境

- 系统 ： centos 7
- Spark 包版本： spark-1.6.1-hadoop-2.6.0
- Scala 版本 scala 2.10.6
- IDE ： IDEA

## 步骤

1. 创建 SBT 工程

2.  引入依赖
	
	编辑 build.sbt 文件，在文件中追加如下代码。
	```sbt
	libraryDependencies ++= Seq(
		"org.apache.spark" % "spark-streaming_2.10" % "1.6.1",
		"org.apache.spark" % "spark-streaming-kafka_2.10" % "1.6.1"
	)
	```

3. 代码编写
	
	- 在 src/main/scala 目录下创建包 Stream
	- 在 Stream 包下创建 Scala 类文件 NetworkWordCount **并选择创建类型为 Object**

       示例代码如下
	```scala
	package Stream
    
    import org.apache.spark.SparkConf
    import org.apache.spark.examples.streaming.StreamingExamples
    import org.apache.spark.storage.StorageLevel
    import org.apache.spark.streaming.{Seconds, StreamingContext}
    
    object NetworkWordCount {
      def main(args : scala.Array[scala.Predef.String]) : scala.Unit = {
    
        if(args.length < 2){
          System.err.println("error")
          System.exit(1)
        }
    
        StreamingExamples.setStreamingLogLevels()
        import org.apache.log4j.{Level,Logger}
    
        Logger.getLogger("spark").setLevel(Level.WARN)
        Logger.getLogger("spark.sql").setLevel(Level.WARN)
        Logger.getLogger("spark.streaming").setLevel(Level.WARN)
    
        val sparkConf = new SparkConf().setAppName("NetworkWordCount")
        val ssc = new StreamingContext(sparkConf, Seconds(1))
    
        val lines = ssc.socketTextStream(args(0),args(1).toInt,StorageLevel.MEMORY_AND_DISK_SER)
    
        val words = lines.flatMap(_.split(" "))
        val wordCounts = words.map(x => (x,1)).reduceByKey(_+_)
    
        wordCounts.print()
    
        ssc.start()
        ssc.awaitTermination()
        ssc.stop()
    
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
	#进入 Spark 目录
	cd /PATH/TO/SPARK
	# 运行 spark 服务
	sbin/start-all.sh
	# 提交任务
	bin/spark-submit  --executor-memory 1g --master spark://127.0.0.1:7077 --class Stream.NetworkWordCount --jars ./lib/spark-examples-1.6.1-hadoop2.6.0.jar [path/to/artifacts.jar] 127.0.0.1 5675	
	```
	
6. 打开另一个终端，运行 netcat 进行数据传输
    - 先安装 netcat 工具
    ```shell
    yum install nc
    ```
    
    - 运行nc工具，连接指定端口
    ```shell
    nc -lk 5675
    ```

9. 在输入任意单词，以空格分割，回车发送

10. 观察运行jar包的终端，观察运行结果