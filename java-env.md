注 ： 本笔记仅对 linux 平台下使用 tar.gz 压缩包安装进行说明

* 下载java
>根据自己的系统下载对应版本的 java sdk 包 [下载地址](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

* 解压压缩包
>本笔记编写时，java sdk 的最新版本为：jdk-8u65-linux-x64.tar.gz
>参考命令
>```shell
>tar -xzf jdk-8u65-linux-x64.tar.gz
>mv jdk-8u65-linux-x64 /usr/local/java
>```

* 配置环境变量
> 参考命令
>```shell
>vim /etc/profile
>```
>新增如下配置
>```shell
>export JAVA_HOME=/usr/local/java
>export JRE_HOME=/usr/local/java/jre
>export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
>export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
>```

* 使配置立刻生效
>参考命令
>```shell
>source /etc/profile
>```

* 测试java环境是否配置成功
>参考命令
>```shell
>java -version
>```

至此，java 环境配置完毕。