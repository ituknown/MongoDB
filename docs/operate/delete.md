# 前言

前面已经介绍了集合（`collection` 或 `table`）如何进行查找、新增、修改和替换操作，现在来看下如何删除集合中的文档数据。

文档数据的删除可以使用 `delete` 语法：

- 删除一条文档数据

```
db.<collection_name>.deleteOne({})
```

- 批量删除文档数据

```
db.<collection_name>.deleteMany({})
```

注意，文档数据删除语法并没有像新增、修改一样的复合语句。

在具体使用之前先来查看一下数据库有那些数据：

```
> db.user_log.find({});

{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "王二", "username" : "wanger" }
{ "_id" : ObjectId("..."), "name" : "麻子", "username" : "lisi" }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan", "age" : 22 }
{ "_id" : ObjectId("..."), "name" : "麻子1" }
{ "_id" : ObjectId("..."), "name" : "zhangshilin" }
{ "_id" : ObjectId("..."), "name" : "linda", "nationality" : "English", "sex" : "girl" }
```

# 单条数据删除

现在基于上述示例中的数据进行删除单条数据操作，比如要删除 `name` 为 `王二` 的数据：

```
> db.user_log.deleteOne({name: '王二'})
{ "acknowledged" : true, "deletedCount" : 1 }

> db.user_log.find({});

{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "麻子", "username" : "lisi" }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan", "age" : 22 }
{ "_id" : ObjectId("..."), "name" : "麻子1" }
{ "_id" : ObjectId("..."), "name" : "zhangshilin" }
{ "_id" : ObjectId("..."), "name" : "linda", "nationality" : "English", "sex" : "girl" }
```

可以看到，名字叫 `王二` 的数据已被删除。现在再来看一个示例，在上面的数据中有五个 `name` 为 `MinGRn` 的用户。现在就来删除该数据：

```
> db.user_log.deleteOne({name: 'MinGRn'})
{ "acknowledged" : true, "deletedCount" : 1 }
```

从输出的日志中可以看到删除了一条数据，并没有将五条数据全部删除。其实删除单条语法的删除依据是删除第一条匹配的数据。

# 批量删除

现在来使用批量删除语法删除 `name` 包含 `麻子` 的所有数据：

```
> db.getCollection('user_log').deleteMany({name: /麻子/})

{ "acknowledged" : true, "deletedCount" : 2 }

> db.user_log.find({});

{ "_id" : ObjectId("5d2ea78cc4ab19ee1ff6d7a2"), "name" : "MinGRn", "username" : "zhanssan" }
{ "_id" : ObjectId("5d2ea900c4ab19ee1ff6d7a5"), "name" : "MinGRn", "username" : "zhanssan" }
{ "_id" : ObjectId("5d2eb39ac4ab19ee1ff6d7a6"), "name" : "MinGRn", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("5d2eb3a6c4ab19ee1ff6d7a7"), "name" : "MinGRn", "username" : "zhanssan", "age" : 22 }
{ "_id" : ObjectId("5d3000213a71adf05eccea09"), "name" : "zhangshilin" }
{ "_id" : ObjectId("5d3069fc6ca9d4da1764b9a9"), "name" : "linda", "nationality" : "English", "sex" : "girl" }
```

# 扩展

集合数据删除除了 `delete` 命令之外还有如下命令：

```
db.<collection_name>.remove({}, <juestOne: boolean>)
```

该命令与 `delete` 类似，第一个条件接受的是条件，第二次参数表示是否全部删除，默认为 `true`。

现在来演示一下：

```
db.user_log.remove({}, true)
```

```
# 查看集合数据
> db.user_log.find();
{ "_id" : ObjectId("5d31d5e46ca9d4da1764b9ab"), "name" : "MinGRn", "username" : "zhanssan" }
{ "_id" : ObjectId("5d31d5e46ca9d4da1764b9ac"), "name" : "MinGRn", "username" : "zhanssan" }
{ "_id" : ObjectId("5d31d5e46ca9d4da1764b9ad"), "name" : "MinGRn", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("5d31d5e46ca9d4da1764b9ae"), "name" : "MinGRn", "username" : "zhanssan", "age" : 22 }
{ "_id" : ObjectId("5d31d5e46ca9d4da1764b9af"), "name" : "zhangshilin" }
{ "_id" : ObjectId("5d31d5e56ca9d4da1764b9b0"), "name" : "linda", "nationality" : "English", "sex" : "girl" }

# 删除一条数据
> db.user_log.remove({}, true);
WriteResult({ "nRemoved" : 1 })

# 再次查看集合数据
> db.user_log.find();
{ "_id" : ObjectId("5d31d5e46ca9d4da1764b9ac"), "name" : "MinGRn", "username" : "zhanssan" }
{ "_id" : ObjectId("5d31d5e46ca9d4da1764b9ad"), "name" : "MinGRn", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("5d31d5e46ca9d4da1764b9ae"), "name" : "MinGRn", "username" : "zhanssan", "age" : 22 }
{ "_id" : ObjectId("5d31d5e46ca9d4da1764b9af"), "name" : "zhangshilin" }
{ "_id" : ObjectId("5d31d5e56ca9d4da1764b9b0"), "name" : "linda", "nationality" : "English", "sex" : "girl" }
```

这就删除了一条数据，现在来演示一下条件批量删除。

```
> db.user_log.remove({name: /MinGRn/});
WriteResult({ "nRemoved" : 3 })

> db.user_log.find();
{ "_id" : ObjectId("5d31d5e46ca9d4da1764b9af"), "name" : "zhangshilin" }
{ "_id" : ObjectId("5d31d5e56ca9d4da1764b9b0"), "name" : "linda", "nationality" : "English", "sex" : "girl" }
```

最后不执行条件，直接执行命令：

```
> db.user_log.remove({});
WriteResult({ "nRemoved" : 2 })
```

所以，`remove` 命令相比较于 `delete` 命令更加暴力，如果不执行任何参数将会将集合中的数据进行全部删除。

所以，慎用！