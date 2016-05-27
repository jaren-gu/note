# maven 使用 oschina 版本库加速项目构建

来源：http://maven.oschina.net/help.html

开源中国特别推出国内的 Maven 中央库，提供高速稳定的网络和服务，为国内 Maven 使用者提供便捷服务。本 Maven 库是从 ibiblio 同步过来的，因为网络等原因，保持每天一次更新。使用开源软件 Nexus 搭建，对外镜像链接地址为：  http://maven.oschina.net/content/groups/public/  


1. 安装 Maven
	- 下载对应系统版本的 maven 包
	- 解压
	- 把 maven 下的 bin 目录路径添加至系统变量中

2. 修改配置

	修改 maven 下的 `conf/settings.xml` 文件。
	```xml
	 <mirrors>
		<mirror>
			<id>nexus-osc</id>
			<mirrorOf>*</mirrorOf>
			<name>Nexus osc</name>
			<url>http://maven.oschina.net/content/groups/public/</url>
		</mirror>
	</mirrors>
	```
	
	如果还需要osc的thirdparty仓库或多个仓库，需要如下修改：
	
	```xml
	<mirrors>
		<mirror>
			<id>nexus-osc</id>
			<mirrorOf>central</mirrorOf>
			<name>Nexus osc</name>
			<url>http://maven.oschina.net/content/groups/public/</url>
		</mirror>
		<mirror>
			<id>nexus-osc-thirdparty</id>
			<mirrorOf>thirdparty</mirrorOf>
			<name>Nexus osc thirdparty</name>
			<url>http://maven.oschina.net/content/repositories/thirdparty/</url>
		</mirror>
	</mirrors>
	```