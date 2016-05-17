#PHP安装kafka扩展

---

**以下操作均在 Centos 6 / 7 下通过，其他发行版未经测试。**

---

1、**安装librdkafka**
 
```bash
cd /usr/local/src #进入安装包存放目录

wget https://github.com/edenhill/librdkafka/archive/master.zip  #下载

mv master.zip librdkafka-master.zip  #修改包名

unzip librdkafka-master.zip  #解压

cd librdkafka-master  #进入安装文件夹

./configure   #配置

make  #编译

make install  #安装
```

---

2、**安装phpkafka**

```bash
cd /usr/local/src  #进入安装包存放目录

wget https://github.com/EVODelavega/phpkafka/archive/master.zip  #下载

mv master.zip phpkafka-master.zip  #修改包名

unzip phpkafka-master.zip   #解压

cd phpkafka-master   #进入安装文件夹

/usr/local/php/bin/phpize  #加载php扩展模块

./configure --enable-kafka --with-php-config=/usr/local/php/bin/php-config   #配置

make  #编译

make install  #安装
```

---

3、**修改php配置文件**

```bash
vi /usr/local/php/etc/php.ini  #打开php配置文件，在最后一行添加下面的代码

extension="kafka.so"

:wq!  #保存退出
```

---

4、**测试**

以下代码，保存为phpinfo.php

```php
<?php

phpinfo();

?>
```
上传到网站目录，查找kafka，如下图所示，说明安装成功！
![php-kafka.png](/public/imgs/php-kafka.png "php-kafka.png")
  
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