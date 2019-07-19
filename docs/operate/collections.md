# 前言

`MongoDB` 中的表也是集合的意思，等同于 `MySQL` 的 `table`。这个容易弄混，后面说某个包统一会说成集合，这里进行声明一下。

# 查看集合

由于 `MongoDB` 中的集合即为表的意思所以如下两个命令都可以查看集合（表）：

```
show collections;
  或
show tables;
```

现在来切换到 `itumate` 数据库来练练该命令：

```
# 查看数据库
> show dbs;
admin    0.000GB
config   0.000GB
itumate  0.000GB
local    0.000GB

# 切换到 itumate 数据库
> use itumate;
switched to db itumate

# 查看集合
> show collections;
user_log

# 查看集合
> show tables;
user_log
```

# 删除集合

集合删除可以使用如下命令：

```
db.<collection_name>.drop();
```

注意，该命令删除后将无法恢复。所以谨慎使用，下面来进行演示一下：

```
# 先查看一下集合
> show collections
user_log

# 删除集合
> db.user_log.drop()
true

# 再查看一下集合
> show collections
```

这就删除成功了！