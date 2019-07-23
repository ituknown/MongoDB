# 前言

`MongoDB` 的 `aggregate` 函数主要适用于数据聚合查询操作，诸如统计平均值，求和等并返回计算后的数据结果。

`aggregate` 聚合命令使用如下所示：

```bash
db.<collection_name>.aggregate([pipeline]);
  或
db.getCollection('<collection_name>').aggregate([pipeline]);
```

- `pipeline` 参数是管道的概念，接受的是数组文档数据。

管道在 `Unix` 和 `Linux` 中一般用于将当前命令的输出结果作为下一个命令的参数。

`MongoDB` 的聚合管道将 `MongoDB` 文档在一个管道处理完毕后将结果传递给下一个管道处理。管道操作是可以重复的。

常用的聚合框架如下所示：

|    框架    | 描述                                                         |
| :--------: | :----------------------------------------------------------- |
| `$project` | 修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。 |
|  `$match`  | 用于过滤数据，只输出符合条件的文档。`$match` 使用 MongoDB 的标准查询操作。 |
|  `$limit`  | 用来限制 MongoDB 聚合管道返回的文档数。                      |
|  `$skip`   | 在聚合管道中跳过指定数量的文档，并返回余下的文档。           |
| `$unwind`  | 将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。 |
|  `$group`  | 将集合中的文档分组，可用于统计结果。                         |
|  `$sort`   | 将输入文档排序后输出。                                       |
| `$geoNear` | 输出接近某一地理位置的有序文档。                             |

# 数据准备

在具体演示之前我们先来创建一个新库： `reduce`，并在库中创建一个新的集合：`user`

```bash
> use reduce;
switched to db reduce

> db.createCollection('user');
{ "ok" : 1 }
```

现在我们来向这个新的集合中增加如下数据：

```bash
db.user.insert({name: '张三', age: 18, addr: '上海徐汇'});
db.user.insert({name: '张三', age: 20, addr: '上海黄浦'});
db.user.insert({name: '李四', age: 32, addr: '浙江杭州'});
db.user.insert({name: '李雷', addr: '美国旧金山'});
db.user.insert({name: '韩梅梅', age: 18, addr: '中国北京'});
db.user.insert({name: 'Bob', age: 24, addr: '美国'});
db.user.insert({name: 'Linda', age: 18, addr: '美国', sex: '女'});
```

数据添加完成后现在就来看下如何基于上述管道的概念做聚合查询操作。

**注意：** 在下述演示的命令中会用到一些基于统计的函数，暂时只需要知道意思即可，后文会做进一步讲解。

# 通过用户分组计数

我们以上面的数据为演示基础，在 [前言](#前言) 中介绍了几个聚合框架，其中 `$group` 就是用于分组的聚合框架。那么这里如果要想基于用户做分组的话就
不能不使用该框架。所以，命令为：

```bash
> db.user.aggregate([{$group: {_id: '$name'}}]);
```

现在来看下为什么要这么写这条命令。在之前介绍了，`aggregate` 聚合函数接受的是一个数组文档表达式。所以，这里我们在数据内部使用文档数据。`$group`
是管道函数，唯一需要讲解的是这个管道对应的文档数据： `{_id: '$name'}`。这里的 `_id` 你可以理解为指定文档的 `key`，是个定值。这个 `key` 指定
的值就是文档数据的 `key`。因为我们这里要统计的是按用户名称分组计数，所以对应的是 `name`。如果要按照地址计数的话这里就要使用 `addr` 了。另外，
我们在 `key` 前面使用了 `$` 符号。这是 MongoDB 的语法，意为指定 key 或者使用函数的意思。后续会介绍有关函数。

现在来执行这条语句，得到的结果如下：

```bash
> db.user.aggregate([{$group: {_id: '$name'}}]);

{ "_id" : "Linda" }
{ "_id" : "张三" }
{ "_id" : "韩梅梅" }
{ "_id" : "Bob" }
{ "_id" : "李雷" }
{ "_id" : "李四" }
```

似乎少了什么？是不是少了具体名称的数值啊？这里就需要借助 `$sum` 函数进行计数了，现在我们来增加该函数：

```bash
> db.user.aggregate([{$group: {_id: '$name', sum: {$sum: 1}}}]);
```

`sum` 是指定的 key，而 `$sum` 是计算总和函数，这里每匹配到一个用户就增加 1 ，所以我们指定的值为 1。现在来看下执行结果：

```bash
> db.user.aggregate([{$group: {_id: '$name', sum: {$sum: 1}}}]);

{ "_id" : "Linda", "sum" : 1 }
{ "_id" : "张三", "sum" : 2 }
{ "_id" : "韩梅梅", "sum" : 1 }
{ "_id" : "Bob", "sum" : 1 }
{ "_id" : "李雷", "sum" : 1 }
{ "_id" : "李四", "sum" : 1 }
```

现在应该大致了解了聚合函数的基本用法，那现在继续进一步讲解。

# 匹配年龄小于 32 的用户并按名称分组计数

现在要求的是先匹配年龄小于 32 的用户，将得到的结果再进行按名称分组计数。

匹配？那是不是应该使用 `$match` 函数呢？先来试一下：

```bash
> db.user.aggregate([{$match: {age: {$lt: 32}}}]);
```

如果你已经看了 [查询操作](../operate/find.md) 那么这条语句就不难理解了，其实这就是一个 `find` 条件查找，现在执行看下结果：

```bash
> db.user.aggregate([{$match: {age: {$lt: 32}}}]);

{ "_id" : ObjectId("..."), "name" : "张三", "age" : 18, "addr" : "上海徐汇" }
{ "_id" : ObjectId("..."), "name" : "张三", "age" : 20, "addr" : "上海黄浦" }
{ "_id" : ObjectId("..."), "name" : "韩梅梅", "age" : 18, "addr" : "中国北京" }
{ "_id" : ObjectId("..."), "name" : "Bob", "age" : 24, "addr" : "美国" }
{ "_id" : ObjectId("..."), "name" : "Linda", "age" : 18, "addr" : "美国", "sex" : "女" }
```

从返回的结果中看到，确实将年龄大于等于 32 岁的用户过滤掉了，现在再来根据这个结果进行分组计数：

```bash
> db.user.aggregate([{$match: {age: {$lt: 32}}}, {$group: {_id: "$name", sum: {$sum: 1}}}]);
```

在 MongoDB 中的管道其实就想到与 MySQL 中的子查询操作，类似于如下：

```
select * from user where id in ( select uid from user_role where role = 1);
```

所以，这里的 MongoDB 语句的意思就是 `$match` 匹配的结果作为 `$group` 的条件，得到的结果如下所示：

```bash
> db.user.aggregate([{$match: {age: {$lt: 32}}}, {$group: {_id: "$name", sum: {$sum: 1}}}]);

{ "_id" : "Bob", "sum" : 1 }
{ "_id" : "韩梅梅", "sum" : 1 }
{ "_id" : "Linda", "sum" : 1 }
{ "_id" : "张三", "sum" : 2 }
```

很完美的结果，这确实是我们想要的。

现在我想要将上面的结果倒叙排序该怎么做？

还用想？肯定使用 `$sort` 框架啊，直接上语句结果：

```bash
> db.user.aggregate([{$match: {age: {$lt: 32}}}, {$group: {_id: "$name", sum: {$sum: 1}}}, {$sort: {sum: -1}}]);

{ "_id" : "张三", "sum" : 2 }
{ "_id" : "Bob", "sum" : 1 }
{ "_id" : "韩梅梅", "sum" : 1 }
{ "_id" : "Linda", "sum" : 1 }
```

# $project 框架说明

这里可能唯一要说明的就是 `$project` 框架了，该框架就相当于在 [查询操作](../operate/find.md) 中介绍的返回指定字段的意思，现在看下：

```bash
> db.user.aggregate([{$project: {name: 1, age: 1}}]);
```

该语句得到的结果如下：

```
{ "_id" : ObjectId("..."), "name" : "张三", "age" : 18 }
{ "_id" : ObjectId("..."), "name" : "张三", "age" : 20 }
{ "_id" : ObjectId("..."), "name" : "李四", "age" : 32 }
{ "_id" : ObjectId("..."), "name" : "李雷" }
{ "_id" : ObjectId("..."), "name" : "韩梅梅", "age" : 18 }
{ "_id" : ObjectId("..."), "name" : "Bob", "age" : 24 }
{ "_id" : ObjectId("..."), "name" : "Linda", "age" : 18 }
```

为什么 `name` 和 `age` 后面的值为 1 这里不做说明，具体见 [查询操作#返回指定字段查询](../operate/find.md#返回指定字段查询)。

# 聚合表达式

在上面使用的 `$sum` 表达式，这里再扩展一下表达式及使用方式：

| 表达式      | 描述                                           | 实例                                                         |
| :---------- | :--------------------------------------------- | :----------------------------------------------------------- |
| `$sum`      | 计算总和。                                     | `db.user.aggregate([{$group : {_id : "$name", num_tutorial : {$sum : "$age"}}}])` |
| `$avg`      | 计算平均值                                     | `db.user.aggregate([{$group : {_id : "$name", num_tutorial : {$avg : "$age"}}}])` |
| `$min`      | 获取集合中所有文档对应值得最小值。             | `db.user.aggregate([{$group : {_id : "$name", num_tutorial : {$min : "$age"}}}])` |
| `$max`      | 获取集合中所有文档对应值得最大值。             | `db.user.aggregate([{$group : {_id : "$name", num_tutorial : {$max : "$age"}}}])` |
| `$push`     | 在结果文档中插入值到一个数组中。               | `db.user.aggregate([{$group : {_id : "$name", country : {$push: "CN"}}}])` |
| `$addToSet` | 在结果文档中插入值到一个数组中，但不创建副本。 | `db.user.aggregate([{$group : {_id : "$name", country : {$addToSet : "CN"}}}])` |
| `$first`    | 根据资源文档的排序获取第一个文档数据。         | `db.user.aggregate([{$group : {_id : "$name", first_addr : {$first : "$addr"}}}])` |
| `$last`     | 根据资源文档的排序获取最后一个文档数据         | `db.user.aggregate([{$group : {_id : "$name", last_addr : {$last : "$addr"}}}])` |

具体事例直接见说明即可，就不做说明了。