---
title: DB MOngo MongoDB基础
date: 2018-11-24 00:00:00
tags: 数据库
categories: [DB, MOngo]
---

## MongoDB 是一个基于分布式文件存储的数据库。由 C++ 语言编写。旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。

MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。

<!-- more -->
![MongoDB](./mongodb.png)

# 0.数据库启动/关闭 🍕
启动mongonDB:
C:\mongo\mongodb\bin >>
``` shell
mongod --dbpath C:\mongo\mongodb\data\db
mongod --dbpath C:\mongo\mongodb\data\db --repair # 异常关闭时重启
```

关闭mongodb:
mongo_shell >>

```
use admin
db.shutdownServer()
```

---

# 1.数据库的导入和导出 🍔
``` shell 
本地:127.0.0.1:27017
线上:xxx.xxx.xx.xxx:xxxxx-u xxxx -p xxxxxxxx

**BSON**
bson格式导出
mongodump -h 127.0.0.1:27017 -d xxxx -c xxxx -o ./
mongodump -h xxx.xxx.xx.xxx:xxxx -d xxxx -c xxxx -u xxxx -p xxxxxxxx -o ./
bson格式导入
mongorestore -h xxx.xxx.xx.xxx:xxxx -d xxxx --dir F:\xxxx -u xxxx -p xxxxxxxx
mongorestore -h 127.0.0.1:27017 -d xxxx--dir F:\xxxx\xxxx

**JSON**
json格式导出
mongoexport -h xxx.xxx.xx.xxx:xxxx -d xxxx-c xxxx -u xxxx -p xxxxxxxx --out name.json
json格式导入
mongoimport -h xxx.xxx.xx.xxx:xxxx -d xxxx -c xxxx --file name.json -u xxxx -p xxxxxxxx
mongoimport -h xxx.xxx.xx.xxx:xxxx -d xxxx -c xxxx (--drop) --file name.json -u xxxx -p xxxxxxxx
```

---

# 2.更新和删除 字段 🍟

**修改** 字段名

``` js
db.getCollection('pydb').update({}, {$rename : {"pop" : "abc"}}, false, true)
db.getCollection('SS_UniversityInfo').update({}, {$rename : {"XuanKaoScrope" : "SubjectScope"}}, false, true)
```

参数说明：
db.collection.update(criteria,objNew,upsert,multi)
**criteria** ：查询条件
**objNew** ：update对象和一些更新操作符
**upsert** ：如果不存在update的记录，是否插入objNew这个新的文档，true为插入，默认为false，不插入。
**multi** ：默认是false，只更新找到的第一条记录。如果为true，把按条件查询出来的记录全部更新


**更新** 字段

```js
db.table.update({}, {$set: {content:"xxx"}}, false,true)
```

**删除** 字段

``` js
db.table.update({},{$unset:{content:""}},false, true)
```

---
## 练习 🍤

* 修改列表里的字典里的字段名

```js
// 把XuanKaoSubjectScrope 字段改成 Scrope
[
    {
        "SubjectScope":[
            {"XuanKaoSubjectScrope":"x1"},
            {"XuanKaoSubjectScrope":"x2"},
            {"XuanKaoSubjectScrope":"x3"},
        ]
    },
    {},
    {}...
]
```

```js
// [未完美解决] 当时采用的方法是导出json再替换文本😂
db.getCollection('SS_UniversityInfo').update(
    {"SubjectScope":{
        "$elemMatch":{
            "XuanKaoSubjectScrope":{
                "$exists":true}}}},
    {"$rename":{
        "$SubjectScope.$.XuanKaoSubjectScrope":"SubjectScope.$.Scrope"}
        },
    false,
    true)
```

``` js
// 现在发现可以写python或js脚本解决
```

---

* 正则寻找不以cn/com/org/net/hk/edu结尾的字段

```js
db.getCollection('SS_UniversityInfo').find({"WebSite":{"$not":/cn$|com$|org$|net$|hk$|edu$/}})

db.getCollection('SS_UniversityInfo').find({"WebSite":{"$regex":"net$|com$"}})
```

* [MongoDB逻辑操作符$or, $and,$not,$nor](https://blog.csdn.net/yaomingyang/article/details/75103480)

---
 

# 3.MogoDB 聚合 🍳
* [菜鸟教程 MongoDB 聚合](http://www.runoob.com/mongodb/mongodb-aggregate.html)

管道在Unix和Linux中一般用于将当前命令的 **输出结果** 作为下一个命令的 **参数** 。
MongoDB的聚合管道将MongoDB文档在一个管道处理完毕后将结果**传递**给下一个管道处理。
管道操作是可以**重复**的。

简单介绍一下聚合中常用的几个参数：

```py
$project：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
$match：用于过滤数据，只输出符合条件的文档。$match使用MongoDB的标准查询操作。
$limit：用来限制MongoDB聚合管道返回的文档数。
$skip：在聚合管道中跳过指定数量的文档，并返回余下的文档。
$unwind：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
$group：将集合中的文档分组，可用于统计结果。
$sort：将输入文档排序后输出。
$geoNear：输出接近某一地理位置的有序文档
```
