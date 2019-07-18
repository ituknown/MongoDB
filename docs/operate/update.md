# 前言

修改操作与新增同样，都是具有复合查询语句，示例如下：

- 修改单条

```
db.<collection_name>.updateOne({query}, {update}, {upsert: <boolean>})
```

- 批量修改

```
db.collection.update(    
	<query>, 
	<update>, 
	{       
		upsert: <boolean>,   
		writeConcern: <document>
	}
)
```

- 复合修改

```
db.collection.update(    
	<query>, 
	<update>, 
	{       
		upsert: <boolean>,   
		multi: <boolean>,  
		writeConcern: <document>
	}
)
```

参数说明：

- `query`：查询条件
- `update`：修改数据
- `upsert`：（可选）如果要修改的数据不存在是否做新增操作，默认 `false`
- `multi`：（可选）是否修改多条，默认 `false`
- `writeConcern`：（可选）抛出异常级别

通常，修改指令如下：

```
db.<collection_name>.update({},{$set: {}})
```

现在就来看下如何使用：

# 修改单条

下面来修改名称（`name`）为 `李四` 的用户，修改为 `麻子`：

```
> db.user_log.update({name: "李四"}, {$set: {name: "麻子"}})
```

在修改之前先看下数据：

```
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "王二", "username" : "wanger" }
{ "_id" : ObjectId("..."), "name" : "李四", "username" : "lisi" }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("..."), "name" : "张三2", "username" : "zhanssan", "age" : 22 }
```

再看下修改后的数据：

```
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "王二", "username" : "wanger" }
{ "_id" : ObjectId("..."), "name" : "麻子", "username" : "lisi" }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("..."), "name" : "张三2", "username" : "zhanssan", "age" : 22 }
```

看到，名字修改成功了。

注意，这里使用的是复合语法，你也可以直接使用 `updateOne`，示例如下：

```
> db.user_log.updateOne({name: "李四"}, {$set: {name: "麻子"}})
```

# 修改新增操作

该操作是如果要修改的数据不存在就执行新增操作，即增加 `upsert` 选项，如下示例：

现在继续修改一下修改语句如下：

```
> db.user_log.update({name: "李四"}, {$set: {name: "麻子1"}}, {upsert: true})
```

这句命令是将 `李四` 修改为 `麻子`，如果在数据库中不存在则做新增操作！

命令执行后的数据如下：

```
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "王二", "username" : "wanger" }
{ "_id" : ObjectId("..."), "name" : "麻子", "username" : "lisi" }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "张三1", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("..."), "name" : "张三2", "username" : "zhanssan", "age" : 22 }
{ "_id" : ObjectId("..."), "name" : "麻子1" }
```

# 批量修改

现在我们将所有名字包含 `张三` 的用户修改为 `MinGRn`，命令如下所示：

```
> db.user_log.update({name: /张三/}, {$set: {name: "MinGRn"}}, {multi: true})
```

现在再来看下数据：

```
{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "王二", "username" : "wanger" }
{ "_id" : ObjectId("..."), "name" : "麻子", "username" : "lisi" }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan" }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan", "age" : 20 }
{ "_id" : ObjectId("..."), "name" : "MinGRn", "username" : "zhanssan", "age" : 22 }
{ "_id" : ObjectId("..."), "name" : "麻子1" }
```

另外，如果觉得这条修改指令比较繁琐你也可以替换成 `updateMany` 命令，如下所示：

```
> db.user_log.updateMany({name: /张三/}, {$set: {name: "MinGRn"}})
```

# 数据替换

`MongoDB` 修改数据主要使用的是 `update` 语法，很显然如果要替换的文档数据特别多但是又不想保留没有修改的数据该怎么办？来看一下另外一个语法：`save`。

`save` 语法与 `update` 语法不同，该语法意为替换。具体语法如下所示：

```
db.<collection_name>.save(    
	<document>,     
	{      
		writeConcern: <document> 
	}  
) 
```

参数说明：

- `document`：文档数据。
- `writeConcern`：（可选）抛出异常的级别。

现在先来想数据库 `user_log` 中新增一条数据，如下：

```
> db.getCollection('user_log').insert({name: "Bob", nationality: "English", age: 18})

> db.getCollection('user_log').find({}).pretty()

# 查询结果
{
	"_id" : ObjectId("5d3069fc6ca9d4da1764b9a9"),
	"name" : "Bob",
	"nationality" : "English",
	"age" : 18
}
```

看到该数据的 `_id` 为 `5d3069fc6ca9d4da1764b9a9`。在 `MongoDB` 中替换数据需要使用索引作为 key，即这里的 `_id` 就是 `MongoDB` 主键索引，
我们需要借助索引来进行数据提交换。现在我们将 `name` 进行修改为 `Linda` 并且增加一个性别属性 `sex` 为 `girl`：

```
> db.getCollection('user_log').save({
    "_id" : ObjectId("5d3069fc6ca9d4da1764b9a9"),
    "name": "linda",
    "nationality": "English",
    "sex": "girl"
})
```

这条语句没有保留之前的 `age` 属性，如果 `save` 语法确实为替换语法那么之前保存的 `age` 应该不存在，反之如果存在意为修改语法，现在执行以下该语句
看看到底是不是替换语法。

语句执行后查询结果如下所示：

```
{
	"_id" : ObjectId("5d3069fc6ca9d4da1764b9a9"),
	"name" : "linda",
	"nationality" : "English",
	"sex" : "girl"
}
```

所以，`save` 语法可以实现数据替换！

# 更新操作符

还记得前面的修改语句？

```
db.user_log.update({name: "李四"}, {$set: {name: "麻子1"}}, {upsert: true})
```

在修改语句中我么使用了一个 `$set` 操作符，其实这个语法表示修改操作，即修改 `$set` 对象中的文档内容，其实这是更新操作符。除了该操作符之外
 `MongoDB` 还有其他多种更新操作符：

## 字段更新操作符

|方法名	|描述|
|:-----:|:---:|
|`$mul`	| |
|`$max`	| |
|`$min`	| |
|`$set`	|设置某一个字段的值。|
|`$inc`	|对一个数字字段的某个 `field` 增加 `value` |
|`$rename`	|字段的重命名|
|`$unset`	|删除字段。|
|`$setOnInsert` |     |	
|`$currentDate`	|    ||

## 数组更新操作符

|方法名	|描述|
|:-----:|:---:|
|`$each` ||
|`$sort` ||
|`$position` ||
|`$pull`	|从数组 `field` 内删除一个等于 `value` 的值。|
|`$push`	|把 `value` 追加到 `field` 里。|
|`$pushAll`	|用法同 `$push` 一样，只是 `$pushAll` 一次可以追加多个值到一个数组字段内。|
|`$addToSet`	|加一个值到数组内，而且只有当这个值不在数组内才增加。|
|`$pullAll`	|用法同 `$pull` 一样，可以一次删除数组内的多个值。|
|`$pop`	|删除数组内的一个值。|