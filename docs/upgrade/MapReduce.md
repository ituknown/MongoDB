# 前言

在 [聚合查询](./aggregation.md) 中主要介绍了 `aggregate` 聚合函数，该函数接受一个聚合框架集合，前一个聚合框架的结果可供后一个聚合使用。事实上，
前一个聚合框架得到的值是后一个聚合框架的参数。

这里再介绍另一种聚合查询方式 -- `Map-Reduce`。

`Map-Reduce` 是一种计算模型，简单的说就是将大批量的工作（数据）分解（MAP）执行，然后再将结果合并成最终结果（REDUCE）。

`MongoDB` 提供的 `Map-Reduce` 非常灵活，对于大规模数据分析也相当实用。

先来看一下具体句法：

```
db.<collection_name>.mapReduce(
   function() {emit(key,value);},  //map 函数
   function(key,values) {return reduceFunction},   //reduce 函数
   {
      out: collection,
      query: document,
      sort: document,
      limit: number
   }
)
```

- `map`：映射函数 (生成键值对序列,作为 `reduce` 函数参数)。
- `reduce`: 统计函数，`reduce` 函数的任务就是将 `key-values` 变成 `key-value`，也就是把 `values` 数组变成一个单一的值 `value`。
- `out`: 统计结果存放集合 (不指定则使用临时集合,在客户端断开后自动删除)。
- `query`： 一个筛选条件，只有满足条件的文档才会调用map函数。（`query`、`limit`、`sort` 可以随意组合）
- `sort`： 和 `limit` 结合的 `sort` 排序参数（也是在发往 `map` 函数前给文档排序），可以优化分组机制
- `limit`： 发往 `map` 函数的文档数量的上限（要是没有 `limit`，单独使用 `sort` 的用处不大）

简化一下上面的语法如下：

```
db.<collection_name>.mapReduce(function(){}, function(key, values){}, {});
```

这里要说下 `Map-Reduce` 与 `Aggregate` 的主要区别，`Map-Reduce` 函数介绍三个参数：

- 第一个参数：该参数是一个 Map 函数，该函数写法如下：

```
function(){
  emit(key, value)
}
```

其中的 `key` 和 `value` 是一个变量，但是 `emit` 却是固定写法，这个函数是 Map 函数。Map 函数调用 `emit(key, value)`, 遍历 `collection`
中所有的记录, 将 `key` 与 `value` 传递给 `Reduce` 函数进行处理。

- 第二个参数：该参数是一个 `Reduce` 函数，也就是做具体运算的函数：

```
function(key, values) {
  return reduceFunction
}
```

该参数的 `key` 是 `Map` 函数的 `key`， 而 `values` 却是对应 `key` 的集合值，下面会具体说明。

- 第三个参数：该参数是一个文档对象。

# 数据准备

先来准备一些数据：

```
for(var i = 0; i < 1000; i++ ) {
  db.user.insert({name: "user1", age: i});
}

for(var i = 0; i < 1000; i++ ) {
  db.user.insert({name: "user2", age: i});
}

for(var i = 0; i < 1000; i++ ) {
  db.user.insert({name: "user3", age: i});
}

for(var i = 0; i < 1000; i++ ) {
  db.user.insert({name: "user4", age: i});
}

for(var i = 0; i < 1000; i++ ) {
  db.user.insert({name: "user5", age: i});
}
```

一共准备了 5k 条数据，一共 5 个同名用户，只是年龄是从 1 - 1000。

现在我要统计同名用户一共有多少个人，那么该统计要怎么写呢？

当然，如果使用 `aggregate` 很容易就写出来了，但这里我们主要看如何使用 `map-reduce` 实现。

`Map-Reduce` 函数中，第一个参数接受的就是映射函数，话句话说我们要统计的数据需要在这里进行指定：用户名和数量！所以写法如下：

```
function () {
  emit (this.name, 1)
}
```

为什么这么写？`this.name` 表示指定的数据库属性名为 `name`，而后面的 1 是数值，表示一个用户的数值为 1。这个值主要用户 `reduce` 函数计算使用。

现在再来写 `reduce` 函数：

```
function (key, values) {
  return Array.sum(values)
}
```

没什么好说的，只是将值进行相加。

最后输出文档与条件，这里的条件我们要求年龄小于 100 的用户：

```
{
  query: {age: {$lt:  100}},
  out: 'name_sum'
}
```

最后，整个函数如下所示：

```
db.user.mapReduce(
  function(){
    emit(this.name, 1)
  },
  function(key, values){
    return Array.sum(values)
  },
  {
    query: {age: {$lt: 100}},
    out: 'name_sum'
  }
).find();
```

查询结果如下：

```
{"_id" : "user1","value" : 100.0}
{"_id" : "user2","value" : 100.0}
{"_id" : "user3","value" : 100.0}
{"_id" : "user4","value" : 100.0}
{"_id" : "user5","value" : 100.0}
```

到此，就简单的使用了 `Map-Reduce` 进行统计计算。

下面再来看一个：按用户分组，统计年龄在 100 - 1000 这段区间内的用户年龄总和。

翻译一下就是，将姓名相同并且年龄在 100 - 1000 的用户的年龄进行相加。

想一下前面一个示例，我们在计算用户数的时候给的值是 1，表示一个 用户。那这么要计算年龄 ...... 当然是给年龄了。

直接贴表达式：

```
db.user.mapReduce(
  function(){
    emit(this.name, this.age)
  },
  function(key, values){
    return Array.sum(values)
  },
  {
    out: 'age_sum',
    query: {age: {$gt: 100, $lt: 1000}}
  }
).find();
```

最后，输出结果如下：

```
{ "_id" : "user1", "value" : 494450 }
{ "_id" : "user2", "value" : 494450 }
{ "_id" : "user3", "value" : 494450 }
{ "_id" : "user4", "value" : 494450 }
{ "_id" : "user5", "value" : 494450 }
```

