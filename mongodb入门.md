# MongoDB基础入门
## 基础知识
### MongoDB简介
* MongoDB是一个可扩展，开源，表结构自由，用C++语言编写的面向文档的数据库。旨在为web应用提供高性能、高可用性的且易扩展的数据存储解决方案。
* MongoDB是最像关系数据库的NoSQL数据库，支持关系数据库的绝大部分查询。MongoDB不支持join语句。
* MongoDB服务端可运行于Linux，Windows或OS X平台，支持32位和64位应用，默认监听端
口是27017。
* MongoDB内存管理依赖于操作系统的自动内存管理机制，通过Map对数据文件进行内存映射，推荐在64位平台上运行，否则32位模式受虚拟内存地址大小的限制，而且运行时支持的最大文件尺寸也只能为2GB。
* MongoDB支持日志功能Journaling，对数据库的CRUD操作都会记录在日志文件中（CRUD操作指，CREATE，READ，UPDATE，DELETE增删改查操作）。
* MongoDB支持副本集和自动分片。
* MongoDB文档结构和SQL结构的对应关系:

|SQL术语/概念|MongoDB术语/概念|解释说明|
|---|:------:|:---|
|database|database|数据库|
|table|collection|数据库表/集合|
|row|document|数据记录行/文档|
|column|field|数据字段/域|
|index|index|索引|
|table joins|/|表连接,MongoDB不支持|
|primary key|primary key|主键,MongoDB自动将_id设置为主键|

### MongoDB安装
#### Windows安装
##### 安装
* MongoDB提供了32位和64位系统的预编译二进制包，可从MongoDB官网下载安装，MongoDB预编译二进制包下载地址：http://www.mongodb.org/downloads。MongoDB2.2 版本后已经
不再支持 Windows XP 系统。
* 版本介绍：
    * MongoDB for Windows 64-bit 适合 64 位的 Windows Server 2008 R2, Windows 7 , 及最新版本的 Window 系统
    * MongoDB for Windows 32-bit 适合 32 位的 Window 系统及最新的 Windows Vista。 32 位系统上 MongoDB 的数据库最大为 2GB
    * MongoDB for Windows 64-bit Legacy 适合 64 位的 Windows Vista, Windows Server 2003, 及 Windows Server 2008 
* 根据系统下载对应的 .msi 文件，双击运行，按操作提示安装即可。

##### 启动
* 创建数据目录，数据目录必须存在，否则启动失败；假设新建目录为D:/mongo_data/db
* 启动命令：`mongod --dbpath=D:/mongo_data/db`；如不指定--dbpath，mongodb默认当
前安装盘符下的mongodb/db目录。
* 将MongoDB服务器作为Windows服务运行请注意，你必须有管理权限才能运行下面的命令。执行以下命令将MongoDB服务器作为Windows服务运行：
```
mongod.exe --bind_ip yourIPadress --logpath "D:/mongo_data/db/mongodb.log" --logappend
 --dbpath "D:/mongo_data/db" --port yourPortNumber --serviceName "YourServiceName" 
 --serviceDisplayName "YourServiceName" --install
```
* 使用配置文件的方式启动，创建mongod.conf，使用`mongod -f mongod.conf`运行
```conf
# 数据目录
dbpath = D:/mongo_data/db
# 端口
port = 27017
# 是否使用journal日志
journal = true
# 日志路径
logpath = D:/mongo_data/db/logs/mongo_start.log
# 是否开启授权
auth = true
```
建议使用配置文件的方式。

##### MongoDB后台管理 Shell
```
mongo ip:port/dbname
```
如不指定ip，端口和dbname，默认链接本机27107的test数据库。

#### Linux安装
##### 安装
```
1，下载
curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.0.6.tgz
2，解压
tar -zxvf mongodb-linux-x86_64-3.0.6.tgz
3，拷贝到指定目录
mv  mongodb-linux-x86_64-3.0.6/ /usr/local/mongodb
```
##### 启动
启动方式和Windows类似，注意调整路径。

### 几个重要的进程介绍(以windows平台为例，linux类似)
#### 1，mongod
mongod.exe是数据库实例进程，是整个mongodb中最核心部分，负责数据库的创建、删除等各项管理工作，运行在服务器端
为客户端提供监听，相当于mysql中的mysqld进程。使用示例：`mongod -f D:/mongo_data/conf/mongod.conf`
#### 2，mongo
mongo是一个与mongod进程进行交互的Javascript Shell进程，它提供了一些交互的接口函数，用于系统管理员对数据库系
统进行管理。使用示例`mongo --port 27017 -username xxx -password xxx -authenticationDatabase admin`
#### 3，mongodump
mongodump提供了一种从mongod实例上创建BSON dump文件的方法，mongorestore则相反，能够利用这些dump文件重建数据库
，常用命令如下：`mongodump --port 27017 --db test --out D:/mongo_data/dump`
#### 4，mongoexport
mongoexport是一个将mongodb数据库实例中的数据导出来生成JSON或者CSV文件的工具。命令格式如下：`mongoexport --port 27017 --db test --out collection mytest --out D:/mongo_data/test.json`
#### 5，mongoimport
mongoimport和mongoexport刚好相反，是一个将CSV或JSON文件内容导入到mongodb实例中的工具，常用命令格式:`mongoimport --port 50000 --db shop --collecition goods --file D:/mongo_data/goods.csv`
#### 6，mongos
mongos是一个在分片中用到的进程，所有应用程序端的查询操作都会先有它分析，然后将查询定位到某一个分片上。它的作用与mongod类似，客户端的mongo与它连接。
#### 7，mongofiles
mongofiles提供了一个操作mongodb分布式文件存储系统的命令行接口，常用命令如下：`mongofile --port 50000 --db mydocs --local D:/mongo_data/docs/C++ Primer.pdf put c++_primer.pdf`
#### 8，mongostat
mongostat提供了一个展示当前正在运行的mongod实例的状态工具。
#### 9，mongotop
mongotop提供一个而分析mongodb实例花在读写数据上的时间的跟踪方法。它提供的统计数据在每一个collection的级别上。

## MongoDB CRUD操作
### 1，插入文档（Create）
mongodb使用insert()或者save()向集合插入文档，基本语法：
```
db.COLLECTION_NAME.insert(document)
```
实例：
```
db.staffinfo.insert({"name":"tdx10274","desc":"very handsome","option":{"age":18,"addr":"wuhan"}})
```
### 2，查询操作（Read）
语法格式:
```
db.COLLECTION_NAME.find(document);
```
可使用`db.COLLECTION_NAME.find(document).pretty()`美化输出。
实例：
```
db.staffinfo.find({"name":"abc"});
```

### 3，更新操作（Update）
语法格式：
```
db.collection_name.update(
	<query>,
	<update>,
	{
		upsert:<boolean>,
		mulit:<boolean>,
		writeConcern:<document>
	}
)
```
参数说明：
* query : update的查询条件，类似sql update查询内where后面的。
* update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
* upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
* multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
* writeConcern :可选，抛出异常的级别

实例：
```
db.staffinfo.update({"name":"tdx10274"},{$set:{"desc":"smart and handsome"}})
```
### 4，删除操作（Delete）
语法格式：
```
db.collection.remove(
	<query>,
	{
		justOne:<boolean>,
		writeConcern:<document>
	}
)
```



## MongoDB高级操作
## MongoDB复制集
## MongoDB分片
## MongoDB监控与管理
## MongoDB工具介绍

