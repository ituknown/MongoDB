# 前言

`MongoDB` 数据库与 `MySQL` 有很大的不同，现在看下基础的命令。

# 查看数据库

查看数据库使用的命令如下：

```
show dbs
```

在 `MySQL` 中则是 `show databases` 这里有些区别，可以理解为 `MongoDB` 库命令命令是 `SQL` 的简版。

# 创建数据库

创建数据库使用的命令如下：

```
use <db_name>
```

优点意外，创建命令也是使用命令。如果想要使用某个数据库同样是该命令，这里需要进行说明一下。

先来看下面的演示：

```
> show dbs;

admin    0.000GB
config   0.000GB
local    0.000GB
```

当前有三个数据库，现在我们来创建一个新的数据库，库名为 `itumate`：

```
# 创建库
> ues itumate
switched to db itumate
```

现在来查看一下当前有哪些库：

```
> show dbs;

admin    0.000GB
config   0.000GB
local    0.000GB
```

从列出的库中并没有刚刚创建的数据库，这是什么原因？

事实上，在 `MongoDB` 中使用 `use` 命令使用数据库时，如果指定的库存在则直接使用，如果不存在你可以理解为仅仅是将这个没有的库的名字暂时 `缓存` 起
来了（事实并非如此）。当你向这个根本不存在的库中新增一个集合或者新增数据时才会检查库是否存在，如果不存在则进行创建。

现在在这个库中创建一个集合，名字为 `user_log`：

```
> db.createCollection('user_log')
{ "ok" : 1 }

> show dbs;
admin    0.000GB
config   0.000GB
itumate  0.000GB
local    0.000GB
```

创建一个集合后才会将这个库真正的创建出来。

现在来演示另外一种方式，新创建一个库：

```
> use temp;
switched to db temp

> show dbs;
admin    0.000GB
config   0.000GB
itumate  0.000GB
local    0.000GB
```

现在我们不使用 `createCollection` 命令创建集合了，而是使用如下命令在集合中新增一条数据：

```
db.temp_log.insert({key: 'temp'});
```

很奇怪，`temp_log` 这个集合根本不存在。先别管这个，先执行再说：

```
> show dbs;
admin    0.000GB
config   0.000GB
itumate  0.000GB
local    0.000GB
temp     0.000GB

> show collections
temp_log
```

说了，这条命令与上面是同样的意思。在新增数据时如果集合不存在则会进行创建。

# 删除库

如果想要删除某个数据库，你需要先进入（使用）这个数据库，然后再这个数据库里执行如下命令即可删除：

```
db.dropDatabase()
```

其中，`db` 指的是当前库。现在来将 `temp` 库进行删除试试：

```
# 先来查看当前数据库
> show dbs;
admin    0.000GB
config   0.000GB
itumate  0.000GB
local    0.000GB
temp     0.000GB

# 切换到 temp 库
> use temp;
switched to db temp

# 删除该库
> db.dropDatabase()
{ "dropped" : "temp", "ok" : 1 }

# 再次查看数据库
> show dbs;
admin    0.000GB
config   0.000GB
itumate  0.000GB
local    0.000GB
```

现在库就被删除了。