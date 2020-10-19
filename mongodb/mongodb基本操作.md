# MongoDB操作

[toc]
### 0. 前言

　　MongoDB 是一款跨平台、面向文档的数据库。用它创建的数据库可以实现高性能、高可用性，并且能够轻松扩展。MongoDB 的运行方式主要基于两个概念：`集合（collection）`与`文档（document）`。
　　集合与文档的关系是：
　　**一条记录就是一个文档，一个集合包含多个文档。**
　　MongoDB和MySQL的对比：

| 关系型数据库 |               MongoDB                |
| :----------: | :----------------------------------: |
|    数据库    |                数据库                |
|      表      |                 集合                 |
|      行      |                 文档                 |
|      列      |                 字段                 |
|      表      |                 Join                 |
|     主键     | 主键（由 MongoDB 提供的默认 key_id） |
　　一条MongoDB记录如下：
```json
{
   _id: ObjectId(7df78ad8902c)
   title: 'MongoDB Overview', 
   description: 'MongoDB is no sql database',
   by: 'tutorials point',
   url: 'http://www.tutorialspoint.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100, 
   comments: [  
      {
         user:'user1',
         message: 'My first comment',
         dateCreated: new Date(2011,1,20,2,15),
         like: 0 
      },
      {
         user:'user2',
         message: 'My second comments',
         dateCreated: new Date(2011,1,25,7,45),
         like: 5
      }
   ]
}     
```
### 1. 操作

#### 1.1 shell 操作

##### `db`  显示当前所在数据库，默认为 test

##### `show dbs`  列出可用数据库

##### `show collections`  列出数据库中可用集合

##### `use  database `  用于切换数据库
　　用户切换数据库，如果目标数据库不存在，则新建之。返回目标数据库。
 hr 
##### `db.dropDatabase()` 删除当前数据库

##### `db.createCollection(name, options)`创建集合

　　`option`是可选的，参数如下：
|字段	|类型	|描述|
|:--:|:--:|:--:|
|capped	|布尔	|（可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。当该值为 true 时，必须指定 size 参数。
|autoIndexID	|布尔|	（可选）如为 true，自动在 _id 字段创建索引。默认为 false。|
|size|	数值	|（可选）为固定集合指定一个最大值（以字节计）。如果 capped 为 true，也需要指定该字段。
|max	|数值|	（可选）指定固定集合中包含文档的最大数量。|

　　但是，一般很少用这种方式，因为插入一条记录的时候，如果集合不存在会自动创建。
```bash
  db.createCollection("mycol", { capped : true, autoIndexID : true, size : 6142800, max : 10000 } )
{ "ok" : 1 }
```
##### `db.collection_name.drop()`删除一个集合

##### `db.collection_name.insert()` 插入一个或者多个(使用数组)集合
　　如果数据库中不存在该集合，那么 MongoDB 会创建该集合，并向其中插入文档。
　　在插入的文档中，如果我们没有指定 _id 参数，那么 MongoDB 会自动为文档指定一个唯一的 ID。
　　_id 是一个 12 字节长的 16 进制数，这 12 个字节的分配如下：
　　![mongodb_id](./mongodb_id.png)
```bash
 db.post.insert([
{
   title: 'MongoDB Overview', 
   description: 'MongoDB is no sql database',
   by: 'tutorials point',
   url: 'http://www.tutorialspoint.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100
},
{
   title: 'NoSQL Database', 
   description: 'NoSQL database doesn't have tables',
   by: 'tutorials point',
   url: 'http://www.tutorialspoint.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 20, 
   comments: [  
      {
         user:'user1',
         message: 'My first comment',
         dateCreated: new Date(2013,11,10,2,35),
         like: 0 
      }
   ]
}
])
```

##### `db.collection_name.find()`查找文档
　　`find()` 方法会以非结构化的方式来显示所有文档。可以使用 `pretty()` 方法进行结构化输出。
　　除了 `find()` 方法之外，还有一个 `findOne()` 方法，它只返回一个文档。
　　
　　**where操作**
|    操作    |          格式           |                       范例                       |      RDBMS中的类似语句       |
| :--------: | :---------------------: | :----------------------------------------------: | :--------------------------: |
|    等于    |       {key:value}       | db.mycol.find({"by":"tutorials point"}).pretty() | where by = 'tutorials point' |
|    小于    |   {key:{\$lt:value}}    |    db.mycol.find({"likes":{$lt:50}}).pretty()    |       where likes   50       |
| 小于或等于 | { key :{\$lte: value }} |   db.mycol.find({"likes":{$lte:50}}).pretty()    |      where likes  = 50       |
|    大于    | { key :{\$gt: value }}  |    db.mycol.find({"likes":{$gt:50}}).pretty()    |       where likes   50       |
| 大于或等于 | { key :{\$gte: value }} |   db.mycol.find({"likes":{$gte:50}}).pretty()    |      where likes  = 50       |
|   不等于   | { key :{\$ne: value }}  |    db.mycol.find({"likes":{$ne:50}}).pretty()    |      where likes != 50       |

　　**And操作**
　　传入多个键，并用逗号(,)分隔它们。
```bash
db.mycol.find({key1:value1, key2:value2}).pretty()
```
　　**Or操作**
　　使用关键字 `$or`
```bash
>db.mycol.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```
##### `db.collection_name.update(criteria,objNew,upsert,multi)`更新文档
>　　**criteria**：查询条件。
>　　**objNew**：update对象和一些更新操作符。
>　　**upsert**：如果不存在update的记录，是否插入objNew这个新的文档，true为插入，默认为false，不插入。
>　　**multi**：默认是false，只更新找到的第一条记录。如果为true，把按条件查询出来的记录全部更新。
```bash
# 把count大于20的class name修改为c3
> db.classes.update({"count":{$gt:20}},{$set:{"name":"c3"}})
```
　　**$set操作：把文档中某个字段field的值设为value**
```bash
# 把name属性为test的第一条记录的age属性值变为23
db.students.update({name:"test"},{$set:{age:23}})
```
　　**$unset：删除某个字段field**
```bash
# 将test的年龄字段删除
db.students.update({name:"test"},{$unset:{age:1}})
```
　　**$inc：对一个数字字段的某个field增加value**
```bash
# 将test的年龄增加5
db.students.update({name:"test"},{$inc:{age:5}})
```
##### `db.collection_name.save(obj)`保存或更新对象
　　obj代表需要更新的对象，如果集合内部已经存在一个和obj相同的"_id"的记录，Mongodb会把obj对象替换集合内已存在的记录，如果不存在，则会插入obj对象。
##### `db.collocation_name.remove()`删除集合中的一个(默认多个，设定justOne为true即可只删除一个)或者多个文档
```bash
> db.collection_name.remove()  # 删除集合中所有文档
> db.mycol.remove({'title':'titles'})  # 删除集合中所有title为titles的集合
> db.student.remove({age:28}, true)  # 删除第一条满足条件的集合
```
##### `db.collection_name.find({},{KEY:1})`是否显示必要的字段
　　`find()` 方法会默认返回所有的字段，要想限制，可以利用 0 或 1 来设置字段列表。1 用于显示字段，0 用于隐藏字段。
```bash
> db.collection_name.find({},{"title":1,_id:0})  # 只显示title字段，不显示_id字段
```
##### `db.collection_name.find().limit(n)`限制输出n条文档
```bash
>db.collection_name.find({},{"title":1,_id:0}).limit(2)  # 只显示title字段，并且显示前两条
```
##### `db.collection_name.find().limit(NUMBER).skip(NUMBER)`offset操作
　　skip默认为0。
##### `db.collection_name.find().sort({KEY:1})`排序
　　`sort()` 方法可以通过一些参数来指定要进行排序的字段，并使用 1 和 -1 来指定排序方式，其中 1 表示升序，而 -1 表示降序。
```bash
>db.collection_name.find({},{"title":1,_id:0}).sort({"title":-1})  # title按照字典顺序倒序排列
```
##### `db.collection_name.ensureIndex({KEY:1})`索引
　　这里的 key 是想创建索引的字段名称，1 代表按升序排列字段值。-1 代表按降序排列。
　　![mongodb_索引](./mongodb_索引.png)
##### `db.collection_name.aggregate(AGGREGATE_OPERATION)`聚合操作
```bash
> db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])
{
   "result" : [
      {
         "_id" : "tutorials point",
         "num_tutorial" : 2
      },
      {
         "_id" : "Neo4j",
         "num_tutorial" : 1
      }
   ],
   "ok" : 1
}
# 相当于 select by_user, count(*) from mycol group by by_user;
```
![mongodb_聚合函数](./mongodb_聚合函数.png)
##### 数据转储
![mongodb_转储](./mongodb_转储.png)