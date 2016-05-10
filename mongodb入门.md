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
-- insert会自动创建collection，删除collection可使用db.COLLECTION.drop()
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
-- 美化输出
db.COLLECTION_NAME.find(document).pretty()
```
和RDBMS查询Where比较：

|操作|格式|范例|RDBMS中类似语句|
|---|:------|:------|:---|
|等于|{key:value}|db.col.find({"by":"age"}).pretty()|where by = "age"|
|小于|{key:{$lt:value}}|db.col.find({"likes":{$lt:50}}).pretty()|where likes<50|
|小于或等于|{key:{$lte:value}}|db.col.find("likes":{$lte:50}).pretty()|where likes<=50|
类似的有大于($gt),大于或等于($gte),不等于($ne)。
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
实例：
```
db.staffinfo.remove({"name":"tdx10274"});
```
## MongoDB复制集
### 概述
MongoDB复制集实现数据库的冗余备份、故障转移，提高了数据库的可用性和安全性。类似MySQL中的备库。

复制集基本结构：	

![Alt text](/resoure/replication.png)


### 复制集配置
#### 主从复制
```
-- 启动主库
monogd --dbpath="../db" -port=27017 --master
-- 启动从库
monogd --dbpath="../db" --port=8888 --slave --source=127.0.0.1:12707 
```
假设已启动端口为5555的mongod服务，如何添加为从库呢？
```
mongo 127.0.0.1:5555
use local
db.sources.insert({"host":"127.0.0.1:27017"})
-- 查询确认主从复制是否添加成功
db.sources.find()
```

#### 副本集
与主从复制的区别：
* 没有特定的主数据库
* 如果主数据库宕机，集群中会推选一个从数据库作为主数据库顶上，具备自动恢复能力
* 集群中包含一个仲裁节点，集群中数据库数目最好为奇数（包含仲裁数据库），方便仲裁
* 
配置示例1(假设replSet的名称为myrep):
```
-- mongod实例1，端口2222
mongod --port=2222 --dbpath="../db" --replSet myrep/127.0.0.1:3333
-- mongod实例2，端口3333
mongod --port=3333 --dbpath="../db" --replSet myrep/127.0.0.1:2222
-- 初始化副本集，mongo 127.0.0.1:2222
db.initiate({"_id":"myrep","members":[{"_id":1,"host":"127.0.0.1:2222"},{"_id":2,"host":"127.0.0.1:3333"}]});
--添加仲裁节点
mongo --port=4444 -dbpath="../db" --repSet myrep/127.0.0.1:2222：
-- 只能在primary节点上操作
db.addArb("127.0.0.1:4444") 
```
仲裁数据节点只参与选举，并没有数据副本。它起到的作用只是当primary节点故障时，能参与到复制集剩下的节点中选出一个新的primary，仲裁节点永远不会变成primary节点。

注册用复制启动成功后，从备库查询报错:`not master and slaveOk=false`。解决办法：从mongo客户端执行 `db.getMongo().setSlaveOk()`。

## MongoDB分片
### 概述
分片技术是解决mongodb数据量大量增长的需求，存储海量数据。将数据分割存在多个分片上，使得数据库可以存储和处理更多的数据。
副本集主要是提高系统的可用性和安全性，决绝故障转移的问题。每个副本都有完整的数据。

分片的基本结构：	
![Alt text](/resoure/shard.jpg)

### 分片配置
假设有2个复制集rs0,rs1，一个配置服务器configdb，一个mongos

#### 复制集rs0
节点配置示例rs0_0.conf，其他节点类似
```
dbpath = D:/mongo_data/db_rs0/data/rs0_0
logpath = D:/mongo_data/db_rs0/logs/rs0_0.log
journal = true
port = 40000
replSet = rs0
```

启动副本集
```
mongod -f D:/mongo_data/config_rs0/rs0_0.conf
mongod -f D:/mongo_data/config_rs0/rs0_1.conf
mongod -f D:/mongo_data/config_rs0/rs0_2.conf
```
初始化副本集
```
-- 初始化副本集，假设主机名为win7
mongo --port 40000
rs.initiate()
-- 添加副本节点
rs.add("win7:40001")
-- 添加仲裁节点
rs.addArb("win7:40002")
```

#### 复制集rs1 
同上，选择不同的端口和文件目录即可。

#### 配置服务器
配置服务器配置示例（其他副本集节点类似，启动方式参见rs0）
```
dbpath = D:/mongo_data/db_configs/data/db_config0
logpath = D:/mongo_data/db_configs/logs/db_config0.log
journal = true
port = 40006
configsvr = true
```
#### mongos路由服务器
示例配置（cfg_mongos.conf）：
```
logpath = D:/mongo_data/mongos/logs/mongos.log
port = 40009
configdb = win7:40006,win7:40007,win7:40008
```
启动mongos服务器：
```
mongos --config D:/mongo_data/mongos/cfg_mongos.conf
```
添加各个分片到集群：
```
mongo --port 40009
sh.addShard("rs0/win7:40000,win7:40001")
sh.addShard("rs1/win7:40003,win7:40004")

-- 查看状态
sh.status()
```

## 权限控制与监控
### 权限控制

mongod启动后，默认并没有开启权限认证功能，即使配置文件中指定`auth=true`。mongod
激活权限控制之后，所有的操作都必须进行权限认证，mongodb采用基于角色的权限控制，一个角色是一组权限的集合，一个权限决定了用户能对数据库进行哪些操作，用户可能具有多个角色。

数据库admin上保存了针对实例上所有数据库的管理用户。常用角色：
* readAnyDatabase角色：针对所有数据库的只读权限
* readWriteAnyDatabase角色：针对所有数据的读写权限
* userAdminAnyDatabase角色：针对所有数据库的用户管理权限
* dbAdminAnyDatabase角色：真多所有数据库的管理权限
* root：相当于以上四中角色的组合，权限最大

### 权限控制示例
* 针对单个数据库的角色,dbOwner相当于单个数据库的管理员
```
db.createUser(
	{
		user:"test",
		pwd:"123456",
		roles:[{role:"dbOwner",db:"staffinfo"}]
	}
)
```
添加用户完成，在通过`mongo --port 27017`登陆，无权限执行操作。必须通过权限认证才能正常执行操作。

* 复制集和集群的权限控制
首先要创建一个最少6个字符的文件，该温江将被部署到复制集中的每一个节点上，文件内容相当于密码，用作复制集各成员间的权限认证。启动时mongod需要加上`--keyFile`指定密码文件的路径。其他同上。


### 监控
* 1，mongostat工具
命令如下：
```
mongostat --port 50000 -u test -p 123456 --authenticationDatabase admin
```

返回字段：
	* inserts：每秒插入次数
	* query：每秒查询次数
	* update：每秒更新次数
	* delete：每秒删除次数
	* flushes：每秒异步刷新到文件次数
	* mapped：映射的数据文件大小
	* vsize：进程所占的虚拟内存大小
	* res：进程时间占用的物理内存大小
	* faults：每秒产生的缺页错误次数	 
* 2，mongotop工具
命令如下：
```
mongotop --port 50000 -u test -p 123456 --authenticationDatabase admin 5
```
每隔5秒返回统计数据。
* 3，数据库命令serverStatus
```
db.serverStatus()
```
返回字段解释：
	* uptime：实例进程已经激活的总时间，单位秒
	* localTime：实例所在服务器的当前时间
	* totalTime：数据库启动后运行的总时间，单位微妙
	* lockTime：获得全局锁的总时间，单位微妙
	* currentQueue：因为锁引起多谢等待队列数
		* readers：等待读锁的操作数
		* writers：等待写锁的操作数
	* activeClients：链接的激活客户端读写操作的总数
		* readers：激活客户端读操作总数
		* writers：激活客户端写操作总数
	* mem：当前内存使用情况
		* bits：目标机器的架构
		* resident：当前使用的物理内存总量，单位MB
		* virtual：mongod进程印象的虚拟内存大小，单位MB
		* supported：系统是否支持可扩展内存
		* mapped：映射数据文件所使用的内存大小，单位MB
		* mappedWithJournal：映射journaling锁使用的内存大小，单位MB


## MongoDB工具介绍
### adminMongo
数据库管理工具。基于Node.js，界面清爽，操作简单。
#### 安装步骤
*  Clone Repository: git clone https://github.com/mrvautin/adminMongo.git && cd adminMongo
* Install dependencies: npm install
* Start application: npm start
* Visit http://127.0.0.1:1234 in your browser
#### 使用方法
访问http://127.0.0.1:1234

### YCSB
NoSQL基准测试工具。

#### 安装
* 源码编译
	* 下载源码解压
	* cd ycsb-0.7.0
	* 全数据库绑定编译 `mvn clean package`
	* 指定mongodb绑定编译 `mvn -pl com.yahoo.ycsb:mongodb-binding -am clean package`

* 下载安装版
下载页面 https://github.com/brianfrankcooper/YCSB/releases/tag/0.7.0，解压tar.gz即可。

#### 使用方法（windows下，linux下更简单）
* 查看帮助
```
python bin/ycsb -h
```
#### 测试
* Load阶段：加载数据
```
python bin/ycsb load mongodb -threads 100 -s -P workloads/workloada -p mongodb.url=mongodb://mongos:28000/ycsb?w=0 > outputLoad.txt
```

* Run阶段：Load完成后，各种场景运行测试
```
python bin/ycsb run mongodb -threads 100 -s -P workloads/workloada -p mongodb.url=mongodb://mongos:28000/ycsb?w=0 > outputRun.txt
```
每次load数据前要把上次测试中产生的数据删除，包括各个分片，配置服务器，mongos等的数据。