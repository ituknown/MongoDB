# 前言

`MongoDB` 新增有两种语法，另外还有一个符合语法，具体如下：

- 新增单条

```
db.<collection_name>.insertOne({})
```

新增单条语法接受的是一个文档数据，即 `JSON` 格式数据。

- 批量新增

```
db.<collection_name>.insertMany([])
```

批量新增语法接受的是一个数组文档数据。

- 复合语法

```
db.<collection_name>.insert({}|[])
```

复合语法接受文档数据以及数组文档数据。注意，两者不能同时使用。

# 新增单条

单条数据的新增操作可以同时使用 `insertOne` 和 `insert` 语法：

```
> db.user_log.insertOne({name: "张三", username: "zhanssan"})
  或者
> db.user_log.insert({name: "张三", username: "zhanssan"})
```

# 批量新增

批量新增数据可以同时使用 `insertMany` 和 `insert` 语法。注意，此时接受的是数组文档数据：

```
> db.user_log.insertMany([{name: "王二", username: "wanger"},{name: "李四", username: "lisi"}])
  或者
> db.user_log.insert([{name: "王二", username: "wanger"},{name: "李四", username: "lisi"}])
```