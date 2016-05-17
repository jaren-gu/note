# spark-streaming 操作实例

本实例使用环境为：
- centos 6.7 final
- spark-1.6.1-hadoop-2.6.0
- scala 2.10.6
- IDEA

---

1. 新建sbt工程

2. 在 src/main/scala 目录下创建包 Stream

3. 在 Stream 包中创建 NetworkWordCount 对象（注意选择创建类型为object），代码如下:
   
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

4. 为项目创建 Artifacts，并打包

5. 移动包文件至 /tmp

6. 启动本地 spark 集群

7. 运行jar包，参考命令

    ```shell
    bin/spark-submit  --executor-memory 1g --master spark://127.0.0.1:7077 --class Stream.NetworkWordCount --jars ./lib/spark-examples-1.6.1-hadoop2.6.0.jar /tmp/spark.jar 127.0.0.1 5675
    ```
    
8. 打开另一个终端，运行 netcat 进行数据传输
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