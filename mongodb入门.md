# MongoDB基础入门
## MongoDB简介
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

## MongoDB安装
### Windows安装
#### 安装
* MongoDB提供了32位和64位系统的预编译二进制包，可从MongoDB官网下载安装，MongoDB预编译二进制包下载地址：http://www.mongodb.org/downloads。MongoDB2.2 版本后已经
不再支持 Windows XP 系统。
* 版本介绍：
    * MongoDB for Windows 64-bit 适合 64 位的 Windows Server 2008 R2, Windows 7 , 及最新版本的 Window 系统。
    * MongoDB for Windows 32-bit 适合 32 位的 Window 系统及最新的 Windows Vista。 32 位系统上 MongoDB 的数据库最大为 2GB。
    * MongoDB for Windows 64-bit Legacy 适合 64 位的 Windows Vista, Windows Server 2003, 及 Windows Server 2008 。
* 根据系统下载对应的 .msi 文件，双击运行，按操作提示安装即可。

#### 启动
* 创建数据目录，数据目录必须存在，否则启动失败；假设新建目录为D:/mongo_data/db
* 启动命令：`mongod --dbpath=D:/mongo_data/db`；如不指定--dbpath，mongodb默认当
前安装盘符下的mongodb/db目录。
* 将MongoDB服务器作为Windows服务运行请注意，你必须有管理权限才能运行下面的命令。执行以下命令将MongoDB服务器作为Windows服务运行：
```
mongod.exe --bind_ip yourIPadress --logpath "D:/mongo_data/db/mongodb.log" --logappend --dbpath "D:/mongo_data/db" --port yourPortNumber --serviceName "YourServiceName" --serviceDisplayName "YourServiceName" --install
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

#### MongoDB后台管理 Shell
```
mongo ip:port/dbname
```
如不指定ip，端口和dbname，默认链接本机27107的test数据库。

### Linux安装
## MongoDB CRUD操作
## MongoDB高级操作
## MongoDB复制集
## MongoDB分片
## MongoDB监控与管理
## MongoDB工具介绍

