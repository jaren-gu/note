# 如何使用 aliyun-oss-php-sdk

官方文档：

- [github](https://github.com/aliyun/aliyun-oss-php-sdk)
- [帮助中心](https://help.aliyun.com/document_detail/32099.html?spm=5176.doc32099.6.368.bi6PTr)

### 使用步骤
 
1.  在 composer.json 中加入对 aliyun-oss-php-sdk 的依赖声明

2. 运行 composer update 更新项目依赖

3. 引入资源
	
	- ThinkPHP 3.2
	
		在入口文件内加入如下语句
	```php
	require_once './composer/autoload.php';
	```
	
	- laravel 5.1
	
		无需操作

4.  使用
	oss-php-sdk 提供了两个常用的类 `ossClient` 和 `OssException`，使用前需先引入。示例代码：
	```php
	<?php
	
	use OSS\OssClient;
	use OSS\Core\OssException;
	
	$accessKeyId = "<您从OSS获得的AccessKeyId>"; ;
	$accessKeySecret = "<您从OSS获得的AccessKeySecret>";
	$endpoint = "<您选定的OSS数据中心访问域名，例如oss-cn-hangzhou.aliyuncs.com>";
	try {
	    $ossClient = new OssClient($accessKeyId, $accessKeySecret, $endpoint);
	} catch (OssException $e) {
	    print $e->getMessage();
	}
	```

#### 以下是官方说明摘录

## 快速使用

### 常用类

| 类名 | 解释 |
|:------------------|:------------------------------------|
|OSS\OssClient | OSS客户端类，用户通过OssClient的实例调用接口 |
|OSS\Core\OssException | OSS异常类，用户在使用的过程中，只需要注意这个异常|

### OssClient初始化

SDK的OSS操作通过OssClient类完成的，下面代码创建一个OssClient对象:

```php
<?php
$accessKeyId = "<您从OSS获得的AccessKeyId>"; ;
$accessKeySecret = "<您从OSS获得的AccessKeySecret>";
$endpoint = "<您选定的OSS数据中心访问域名，例如oss-cn-hangzhou.aliyuncs.com>";
try {
    $ossClient = new OssClient($accessKeyId, $accessKeySecret, $endpoint);
} catch (OssException $e) {
    print $e->getMessage();
}
```

### 文件操作

文件(又称对象,Object)是OSS中最基本的数据单元，您可以把它简单地理解为文件，用下面代码可以实现一个Object的上传：

```php
<?php
$bucket = "<您使用的Bucket名字，注意命名规范>";
$object = "<您使用的Object名字，注意命名规范>";
$content = "Hello, OSS!"; // 上传的文件内容
try {
    $ossClient->putObject($bucket, $object, $content);
} catch (OssException $e) {
    print $e->getMessage();
}
```

### 存储空间操作

存储空间(又称Bucket)是一个用户用来管理所存储Object的存储空间,对于用户来说是一个管理Object的单元，所有的Object都必须隶属于某个Bucket。您可以按照下面的代码新建一个Bucket：

```php
<?php
$bucket = "<您使用的Bucket名字，注意命名规范>";
try {
    $ossClient->createBucket($bucket);
} catch (OssException $e) {
    print $e->getMessage();
}
```

### 返回结果处理

OssClient提供的接口返回返回数据分为两种：

* Put，Delete类接口，接口返回null，如果没有OssException，即可认为操作成功
* Get，List类接口，接口返回对应的数据，如果没有OssException，即可认为操作成功，举个例子：

```php
<?php
$bucketListInfo = $ossClient->listBuckets();
$bucketList = $bucketListInfo->getBucketList();
foreach($bucketList as $bucket) {
    print($bucket->getLocation() . "\t" . $bucket->getName() . "\t" . $bucket->getCreatedate() . "\n");
}
```

上面代码中的$bucketListInfo的数据类型是 `OSS\Model\BucketListInfo`
	

