# 前言

在 `MongoDB` 中数据查询可以使用如下两个命令：

```
# 使用 collection_name
db.<collection_name>.find({})

# 利用 getCollection('<collcetion_name>')
db.getCollection('<collcetion_name>').find({})
```

两个命令的 `find` 指令是相同的，后面以 `find` 做说明，

`find` 指令如下所示：

```
db.<collection_name>.find({})
```

**注：** `find({})` 接受的是一个文档数据对象，即 `JSON` 格式的数据。

先进入 `user_log` 数据库看下现有的数据：

```
# 查询所有数据
> db.user_log.find({})

# 查询结果
{ "_id" : ObjectId("..."), "name" : "MinGRn", "age" : 18, "email" : "MinGRn97@gmail.com" }
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "王二", "username" : "wanger" }
{ "_id" : ObjectId("..."), "name" : "李四", "username" : "lisi" }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("..."), "name" : "张三2", "username" : "zhanssan", "age" : 22 }
```

下面就以这些命令为基础进行查询操作！

# 指定名称查询

- 查询名称（`name`）叫 `张三` 的用户：

```
> db.user_log.find({name: "张三"})

# 查询结果
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
```

该查询等同于 `MySQL`：

```sql
select * from user_log where name = '张三';
```

# 模糊查询

- 查询名称（`name`）包含 `张三` 的用户，即模糊查询：

```
> db.user_log.find({name: /张三/})

# 查询结果
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("..."), "name" : "张三2", "username" : "zhanssan", "age" : 22 }
```

该查询等同于 `MySQL`：

```sql
select * from user_log where name like '%张三%';
```

- 查询名称（`name`）以 `张` 开头的用户

```
> db.user_log.find({name: /^张三/})

# 查询结果
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("..."), "name" : "张三2", "username" : "zhanssan", "age" : 22 }
```

该查询等同于 `MySQL`：

```sql
select * from user_log where name like '%张';
```

- 查询名称（`name`）以 `三` 结尾的用户

```
> db.user_log.find({name: /三$/})

# 查询结果
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
```

该查询等同于 `MySQL`：

```sql
select * from user_log where name like '三%';
```

# 区间查询

在 `MongoDB` 中区间段查询不像 `MySQL` 可以直接使用 `>`、`<` 等操作，需要使用转义符，如下所示：

  - `$gt`：大于

  - `$gte`：大于等于

  - `$lt`：小于

  - `$lte`：小于等于
  
现在就来看下如何使用：

## 查询年龄大于 18 的用户
```
> db.user_log.find({age: {$gt: 18}})

# 查询结果
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("..."), "name" : "张三2", "username" : "zhanssan", "age" : 22 }
```

该查询等同于 `MySQL`：

```sql
select * from user_log where age > 18;
```


## 查询年龄大于 18 且小于 22 的用户

```
> db.user_log.find({age: {$gt: 18, $lt: 22}})

# 查询结果
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan", "age" : 20 }
```

该查询等同于 `MySQL`：

```sql
select * from user_log where age > 18 and age < 22;
```

# 或者关系查询

或者关系查询使用指令 `$or`，如下所示：

```
db.<collection_name>.find({$or: []})
```

- 查询年龄为 20 和 20 的用户：

```
> db.user_log.find({$or: [{age: 20}, {age: 22}]})

# 查询结果
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("..."), "name" : "张三2", "username" : "zhanssan", "age" : 22 }
```

该查询等同于 `MySQL`：

```sql
select * from user_log where age = 18 or age = 22;
```

# 单表复合查询

- 查询年龄（`age`）大于 18 且名称（`name`）以 `2` 结尾的用户：

```
> db.user_log.find({age: {$gt: 18}, name: /2$/})

# 查询结果
{ "_id" : ObjectId("..."), "name" : "张三2", "username" : "zhanssan", "age" : 22 }
```

该查询等同于 `MySQL`：

```sql
select * from user_log where age > 18 and name like '2%';
```

# 返回指定字段查询

`MongoDB` 条件查询指令一般如下：

```
db.<collection_name>.find({})
```

不过，如果想要返回指定列的字段怎办？

```
db.<collection_name>.find({query}, {field})
```

- `query`：查询条件
- `field`：返回字段

可以看到，现在在查询语句中多了一个选项，第二个选项就是要返回的字段！对于要返回的字段而言需要使用如下选项：

- 返回字段：
  - 等于 0 表示不返回
  - 大于 0 表示返回，一般会使用 1


## 返回指定字段

这里以名称（`name`）字段为例

```
> db.user_log.find({},{name: 1})

# 查询结果
{ "_id" : ObjectId("..."), "name" : "MinGRn" }
{ "_id" : ObjectId("..."), "name" : "张三" }
{ "_id" : ObjectId("..."), "name" : "张三" }
{ "_id" : ObjectId("..."), "name" : "王二" }
{ "_id" : ObjectId("..."), "name" : "李四" }
{ "_id" : ObjectId("..."), "name" : "张三1" }
{ "_id" : ObjectId("..."), "name" : "张三1" }
{ "_id" : ObjectId("..."), "name" : "张三2" }
```

该查询等同于 `MySQL`：

```sql
select name from user_log;
```

## 不返回指定字段

这里以名称（`name`）字段为例

```
> db.user_log.find({}, {name: 0})

# 查询结果
{ "_id" : ObjectId("..."), "age" : 18, "email" : "MinGRn97@gmail.com" }
{ "_id" : ObjectId("..."), "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "username" : "wanger" }
{ "_id" : ObjectId("..."), "username" : "lisi" }
{ "_id" : ObjectId("..."), "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("..."), "username" : "zhanssan", "age" : 22 }
```

该查询等同于 `MySQL`：

```sql
select *(不包含 name) from user_log;
```

# 排序

排序使用 `sort` 指令，如下所示：

```
db.<collection_name>.find({}).sort({})
```

- `sort`：排序
  - `-1`：降序
  - `1`：升序

## 按年龄降序

```
> db.user_log.find({}).sort({age: -1})

# 查询结果
{ "_id" : ObjectId("..."), "name" : "张三2", "username" : "zhanssan", "age" : 22 }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "age" : 18, "email" : "MinGRn97@gmail.com" }
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "王二", "username" : "wanger" }
{ "_id" : ObjectId("..."), "name" : "李四", "username" : "lisi" }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan" }
```

该查询等同于 `MySQL`：

```sql
select * from user_log order by age desc;
```

## 按年龄升序

```
> db.user_log.find({}).sort({age: 1})

# 查询结果
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "王二", "username" : "wanger" }
{ "_id" : ObjectId("..."), "name" : "李四", "username" : "lisi" }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "age" : 18, "email" : "MinGRn97@gmail.com" }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("..."), "name" : "张三2", "username" : "zhanssan", "age" : 22 }
```

该查询等同于 `MySQL`：

```sql
select * from user_log order by age asc;
```

# 分页查询

分页查询可以使用 `limit` 和 `skip` 两个命令组合使用，其中 `limit` 单独使用表示查询 `N` 条数据，与 `skip` 复合使用则表示从第 `N` 条开始向
下查询 `Y` 条数据，指令如下所示：

```
db.<collection_name>.find({}).limit(N).skip(Y)
```

我们先按照年龄降序查询全部数据：

```
> db.user_log.find({}).sort({age: -1})

# 查询结果
{ "_id" : ObjectId("..."), "name" : "张三2", "username" : "zhanssan", "age" : 22 }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "age" : 18, "email" : "MinGRn97@gmail.com" }
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "王二", "username" : "wanger" }
{ "_id" : ObjectId("..."), "name" : "李四", "username" : "lisi" }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan" }
```

下面就利用这些数据进行查询操作：

## 按年龄降序查询前3条数据

```
> db.user_log.find({}).sort({age: -1}).limit(3)

# 查询结果
{ "_id" : ObjectId("..."), "name" : "张三2", "username" : "zhanssan", "age" : 22 }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "age" : 18, "email" : "MinGRn97@gmail.com" }
```

该查询等同于 `MySQL`：

```sql
select * from user_log order by age desc limit 3;
```

- 按年龄降序查询从第三条开始往后查询3条数据

```
> db.user_log.find({}).sort({age: -1}).limit(3).skip(3)

# 查询结果
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "王二", "username" : "wanger" }
```

该查询等同于 `MySQL`：

```sql
select * from user_log order by age desc limit 3, 3;
```

# distinct 去重查询

`distinct` 指令同 `MySQL`，不同的是，只返回具体列的字段，如下所示：

```
> db.user_log.distinct("name")

# 查询结果
[ "MinGRn", "张三", "王二", "李四", "张三1", "张三2" ]
```

# 查询一条数据

查询一条数据命令与 [查询命令](#前言) 相同，只是查询指令不同，具体参考前面即可：

```
db.<collection_name>.findOne({query}, {field})
```

# 条件计数

条件计数语句基于 `find()` 条件。查询命令如下所示：

```bash
db.<collection_name>.find({}).count();
  # 或者
db.getCollection('<collection_name>').find({}).count();
```

- 示例

在做计数之前先查看一下数据库已有数据：

```bash
$ db.getCollection('user_log').find({});

# 查询结果
{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan", "age" : 22 }
{ "_id" : ObjectId("..."), "name" : "zhangshilin" }
{ "_id" : ObjectId("..."), "name" : "linda", "nationality" : "English", "sex" : "girl" }
```

现在我们基于这些数据进行查询计数，在之前先统计一下一共有多少条数据：

```bash
$ db.getCollection('user_log').count();
  # 或者使用如下语句,因为没有条件两个语句都是可以的
$ db.getCollection('user_log').find({}).count();

# 查询结果
6
```

这说明一共有六条数据，现在我们来使用条件统计一下 `name` 包含 `MinGRn` 的用户一共多少个：

```bash
$ db.getCollection('user_log').find({name: /MinGRn/}).count();

# 查询结果如下：
4
```

条件计数查询基本就这些。