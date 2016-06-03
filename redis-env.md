#redis 安装配置

以下操作仅在 Centos 6 / 7 中通过测试，其他发行版未进行测试。

[redis 官网](http://redis.io)


### 安装步骤

1. 安装依赖
	redis 需要依赖 tcl 8.5 或者更高的版本，所以先安装 tcl。
	```bash
	yum -y install tcl
	``` 

2. 下载 redis
	- 使用wget
	```bash
	wget http://download.redis.io/releases/redis-3.2.0.tar.gz
	```
	- 进入官网下载
	[官网下载地址](http://redis.io/download)

3. 解压 redis
	```bash
	tar xzf redis-3.2.0.tar.gz
	```
	
4. 编译 redis
	```bash
	cd redis-3.2.0/
	make && make test
	```
	
5. 安装 redis
	如果上述操作没有报错，就可以正式安装 redis 了。
	```bash
	make install
	```

6. 配置 redis
	```bash
	cd redis.conf /etc/
	vim /etc/redis.conf
	```
	修改	`daemonize no` 为 `daemonize yes` 让 redis 以后台模式运行。

7. 部分配置参数介绍
> daemonize：是否以后台daemon方式运行
> 
> pidfile：pid文件位置
> 
> port：监听的端口号
> 
> timeout：请求超时时间
> 
> loglevel：log信息级别
> 
> logfile：log文件位置
> 
> databases：开启数据库的数量
> 
> save * *：保存快照的频率，第一个*表示多长时间，第三个*表示执行多少次写操作。在一定时间内执行一定数量的写操作时，自动保存快照。可设置多个条件。
> 
> rdbcompression：是否使用压缩
> 
> dbfilename：数据快照文件名（只是文件名，不包括目录）
> 
> dir：数据快照的保存目录（这个是目录）
> 
> appendonly：是否开启appendonlylog，开启的话每次写操作会记一条log，这会提高数据抗风险能力，但影响效率。
> 
> appendfsync：appendonlylog如何同步到磁盘（三个选项，分别是每次写都强制调用fsync、每秒启用一次fsync、不调用fsync等待系统自己同步）