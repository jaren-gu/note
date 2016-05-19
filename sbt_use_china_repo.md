#sbt使用国内源

---

此文参考[原文地址](https://segmentfault.com/a/1190000002474507)

---

因为某些原因，国内使用 sbt 搭建项目出奇的慢。但开源中国为开发者提供了国内的 sbt 源，使用方法也比较简单。

1.  打开 .sbt 目录
	- linux 用户
	```bash
	vim ~/.sbt/
	```
	
	- windows 用户
	微标键+R 运行如下命令
	```bash
	%USERPROFILE%\.sbt
	```
	
2. 编辑 repositories 文件
	如果不存在则创建，编辑内容如下
	- 配置国内代理库
	```bash
	[repositories]
    local
    oschina:http://maven.oschina.net/content/groups/public/ 
	```


	- 兼容 Ivy 路径布局
	```bash
	[repositories]
	local
	oschina:http://maven.oschina.net/content/groups/public/ 
	oschina-ivy:http://maven.oschina.net/content/groups/public/, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext] 
	```
保存退出，重启 sbt 即可。