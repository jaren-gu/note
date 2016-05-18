#PHP安装kafka扩展

---

**以下操作均在 Centos 6 / 7 下通过，其他发行版未经测试。**

---

1、**安装 librdkafka**
 
```bash
#下载
wget https://github.com/edenhill/librdkafka/archive/master.zip

#修改包名
mv master.zip librdkafka-master.zip

#解压并进入安装文件夹
unzip librdkafka-master.zip && cd librdkafka-master

#配置
./configure

#编译安装
make && make install
```

---

2、**安装 kafka 扩展**

- EVODelavega / phpkafka
	```bash
	#下载
	wget https://github.com/EVODelavega/phpkafka/archive/master.zip
	
	#修改包名
	mv master.zip phpkafka-master.zip
	
	#解压并进入安装文件夹
	unzip phpkafka-master.zip && cd phpkafka-master   
	
	#加载php扩展模块
	/usr/local/php/bin/phpize
	
	#配置
	./configure --enable-kafka --with-php-config=/usr/local/php/bin/php-config
	
	#编译安装
	make && make install
	```
- arnaud-lb / php-rdkafka
	```bash
	#下载
	wget https://github.com/arnaud-lb/php-rdkafka/archive/master.zip
	
	#解压并进入安装文件夹
	unzip php-rdkafka-master.zip && cd php-rdkafka-master   
	
	#加载php扩展模块
	/usr/local/php/bin/phpize
	
	#配置
	./configure --with-php-config=/usr/local/php/bin/php-config
	
	#编译安装
	make && make install
	```

---

3、**修改php配置文件**

打开php配置文件，在最后一行添加下面的代码
```bash
#EVODelavega/phpkafka
extension="kafka.so"

#arnaud-lb/php-rdkafka
extension=rdkafka.so
```

---

4、**测试**

以下代码，保存为phpinfo.php

```php
<?php

phpinfo();

?>
```
上传到网站目录，访问并查找kafka，如下图所示，说明安装成功！

- EVODelavega/phpkafka
![php-kafka.png](/public/imgs/php-kafka_1.png "php-kafka_1.png")

- arnaud-lb/php-rdkafka
![php-kafka.png](/public/imgs/php-kafka_2.png "php-kafka_2.png")
  
  ---
  
5、**如果出现如下错误**
```bash
NOTICE: PHP message: PHP Warning: PHP Startup: Unable to load dynamic library '/usr/lib64/php/modules/kafka.so' - librdkafka.so.1: cannot open shared object file: No such file or directory in Unknown on line 0
```
可以执行以下代码解决（[参考链接](https://github.com/salebab/phpkafka/issues/6)）

```bash
echo '/usr/local/lib' > /etc/ld.so.conf.d/kafka.conf

echo 'export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/lib64/php/modules:/usr/local/lib"' >> /etc/profile

ldconfig
```