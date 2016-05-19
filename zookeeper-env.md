# zookeeper 是什么

> zookeeper 是 hadoop 下的一个子项目，是一个针对大型分布式系统的可靠的协调系统 。
> zookeeper 提供了 配置维护、名字服务、分布式同步、组服务 等功能。

# zookeeper 运作原理

> zookeeper 通过 Zab（Zookeeper Atomic Broadcast） 协议保持集群间的数据一致性。  
> Zab 协议包括两个阶段：Leader Election 和 Atomic Broadcast 。  
>
>> Leader Election
>
> * 此阶段集群内会选举出一个 leader，余下机器则会成为 follower。leader 会通过 broadcast 通知所有 follower ，当大部分机（> 1/2）器完成了与 leader 的状态同步后，Leader Election 阶段结束。  
>
> * 当 leader 失去大多数 follower 时，集群会再次进入 Leader Election 阶段并选举出新的 leader ，使集群回到正确的状态。  
> 
>> Atomic Broadcast
>
> * 此阶段 leader 会通过 broadcast 与 follower 通讯，保证 leader 与 follower 具有相同的系统状态。

# zookeeper 搭建
>* 搭建java运行环境
>>可参考我的另一篇笔记
>> [java运行环境配置](https://github.com/jaren-gu/note/blob/master/java_env.md)
>
>* 下载 zookeeper 
>> 下载地址 : [http://mirror.bjtu.edu.cn/apache/zookeeper/stable](http://mirror.bjtu.edu.cn/apache/zookeeper/stable)  
>> 这是国内的镜像源 , 也可以到官网下载 , 此笔记编写时 , zookeeper 稳定版本为 3.4.6  
>> 参考命令 :   
>> ```shell
>>wget http://mirror.bjtu.edu.cn/apache/zookeeper/stable/zookeeper-3.4.6.tar.gz 
>>```
>
> * 解压 zookeeper：  
>> 参考命令：  
>>```shell
>>tar -xf zookeeper-3.4.6.tar.gz
>>cp zookeeper-3.4.6 /usr/local/zookeeper
>>rm -f zookeeper-3.4.6.tar.gz
>>```
>
>* 配置 zookeeper：  
>>参考命令：
>>```shell
>>#创建相应目录，注意目录的权限
>>mkdir /tmp/zookeeper
>>mkdir /tmp/zookeeper/data
>>mkdir /tmp/zookeeper/log
>>
>>cp /usr/local/zookeeper/conf/zoo_sample.cfg zoo.cfg
>>vim /usr/local/zookeeper/conf/zoo.cfg
>>```
>>参考配置：
>>```shell
>>tickTime=2000 #zookeeper 服务器心跳时间，单位为ms
>>initLimit=10  #投票选举新 leader 的初始化时间
>>syncLimit=5  #leader 与 follower 心跳检测最大容忍时间，响应超过 tickTime * syncLimit，认为 leader 丢失该 follower
>>clientPort=2181 #端口
>>dataDir=/tmp/zookeeper/data #数据目录
>>dataLogDir=/tmp/zookeeper/log #日志目录
>>```
>
>* 配置zookeeper日志
>>参考命令：
>>```shell
>>vim /usr/local/zookeeper/bin/zkEnv.sh
>>```
>>参考配置：
>>```shell
>>if [ "x${ZOO_LOG_DIR}" = "x" ]  
>>then  
>>    ZOO_LOG_DIR="/tmp/zookeeper/log"  
>>fi  
>>  
>>if [ "x${ZOO_LOG4J_PROP}" = "x" ]  
>>then  
>>    ZOO_LOG4J_PROP="INFO,ROLLINGFILE"  
>>fi 
>>```
>
> *  配置环境变量 : 
>> 参考命令：
>>```shell
>>vim /etc/profile
>>```
>>新增如下配置：
>>```shell
>>export ZOOKEEPER_INSTALL=/usr/local/zookeeper  
>>export PATH=$PATH:$ZOOKEEPER_INSTALL/bin  
>>```
>
>* 运行zookeeper
>>至此，zookeeper搭建完毕，可以尝试运行zookeeper
>>参考命令：
>>```shell
>>cd /usr/local/zookeeper
>>bin/zkServer.sh start conf/zoo.cfg #启动zookeeper服务，读取配置文件为conf/zoo.cfg
>>bin/zkServer.sh status conf/zoo.cfg #查看指定配置的zookeeper的运行状态
>>```
>可以看到当前 zookeeper 运行状态为 Mode：standalone , 至此，zookeeper单机运行配置完成。