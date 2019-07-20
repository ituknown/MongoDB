# 前言

在各平台安装 `MongoDB` 时是不是发现所有的服务启动使用的端口都是 `27017`，数据存储的目录都是 `/data/db` 并且使用客户端连接时
不需要任何验证信息。

不管从开发角度还是运维角度，都存储数据安全性问题。现在我们来看下如何使用配置文件自定义启动配饰。

# 配置文件

在 `MongoDB` 的安装目录下是没有配置文件的，所以如果要使用配置文件就需要手动创建一个，这里基于 `Mac` 做说明。另外 `Windows` 和
`Linux` 的使用方式都是相同的。

笔者 `MongoDB` 的安装目录为 `/usr/local/mongodb`，所以就直接在二进制包同级目录创建一个配置文件，该配置文件可以放置在任意目录。
配置文件的名称为 `<config-name>.conf`，必须以 `.conf` 结尾！

```bash
$ cd /usr/local/mongodb/bin/
$ sudo touch mongod.conf
```

配置文件创建完成了，那配置文件中的内容可以有哪些呢？别急，现在命令终端输入如下命令：

```bash
$ mongod --help
```

该命令是 `MongoDB` 帮助命令，该命令会列举所有的可选参数。这些参数就是可选的配置属性！

在 Mac 下，笔者以表格的形式列举了部分配置信息：

| 属性        | 示例                | 描述信息                                                     |
| ----------- | ------------------- | ------------------------------------------------------------ |
| `port`      | `27017`             | 启动端口信息                                                 |
| `bind_ip`   | `0.0.0.0`           | 运行绑定的端口信息，注意如果绑定的`ip`为 `127.0.0.1`则其他主机无法连接到该服务。默认为 `0.0.0.0`，即允许所有主机连接 |
| `dbpath`    | `/path`             | 数据存储目录                                                 |
| `logpath`   | `/logs/mongodb.log` | 日志输出文件                                                 |
| `logappend` | `true`              | 是否启动动态日志打印，即在原日志文件中继续输出日志           |
| `quiet`     | `true`              | 是否只输出重要日志信息                                       |
| `syslog`    | `true`              | 是否开启系统日志，如果已经指定了 `logpath` 则不允许开启系统日志 |
| `fork`      | `true`              | 是否启用后台运行服务                                         |
| `install`   | `true`              | 是否安装服务，即已服务的形式进行安装（该参数仅适用于 `windows` 环境） |
| `auth`      | `true`              | 开启安全认证信息                                             |


注意，可选的配置很多。你只需要在命令终端中输入 `mongod --help` 即可列出所有的配置信息，根据实际需要选择即可。

下面来看下如何使用这些配置，如这里笔者想要修改服务启动端口号，以及修改数据、日志输出信息并且后台运行。文件内容如下所示：

```profile
# Net
port=27016
bind_ip=0.0.0.0

# Auth
# auth=true

# Data And Log
dbpath=/Users/mingrn/Mongodb/data
logpath=/Users/mingrn/Mongodb/log/mongodb.log
logappend=true
quiet=true

# Syslog, (Yes Or No)|(true Or False) | (On Or Off)
# Not: syslog is not allowed when logpath is specified
# syslog=no

# Run With Daemon
fork=true
```

配置信息保存后，在命令终端中使用命令指定配置文件启动服务。

```bash
# 使用 --config 参数指定配置文件
$ mongod --config /path/mongod.conf

# 或者使用 -f 参数,该参数时 --config 参数的简版
$ mongod -f /path/mongod.conf
```

现在启动服务：

```bash
$ sudo mongod --config /usr/local/mongodb/bin/mongod.conf 

# 如果你的输出信息如下所示即表示启动成功了
about to fork child process, waiting until server is ready for connections.
forked process: 1834
child process started successfully, parent exiting
```

现在使用 `mongo` 客户端连接，`mongo` 客户端也有需要可选参数，同样可以使用如下命令进行查询所有的可选参数：

```bash
$ mongo --help
```

另外，`mongo` 如果不指定任何参数信息默认连接主机的 `27017` 端口。由于这里在配置文件中修改了端口信息，不指定参数将连接失败，如下所示：

```bash
$ mongo

# 如下是输出的错误信息
MongoDB shell version v4.0.10
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
2019-07-20T14:57:26.716+0800 E QUERY    [js] Error: couldn't connect to server 127.0.0.1:27017, connection attempt failed: SocketException: Error connecting to 127.0.0.1:27017 :: caused by :: Connection refused :
connect@src/mongo/shell/mongo.js:344:17
@(connect):2:6
exception: connect failed
```

现在来使用 `--port` 指定 `27016` 端口：

```bash
$ mongo --port 27016

# 输出信息如下，表示连接成功
MongoDB shell version v4.0.10
connecting to: mongodb://127.0.0.1:27016/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("0629deb7-1fe3-4860-830f-a6a186748021") }
MongoDB server version: 4.0.10
Server has startup warnings: 
2019-07-20T14:56:39.195+0800 I CONTROL  [initandlisten] 
2019-07-20T14:56:39.196+0800 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2019-07-20T14:56:39.196+0800 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2019-07-20T14:56:39.196+0800 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
2019-07-20T14:56:39.196+0800 I CONTROL  [initandlisten] 
2019-07-20T14:56:39.196+0800 I CONTROL  [initandlisten] 
2019-07-20T14:56:39.196+0800 I CONTROL  [initandlisten] ** WARNING: soft rlimits too low. Number of files is 256, should be at least 1000
---
Enable MongoDB's free cloud-based monitoring service, which will then receive and display
metrics about your deployment (disk utilization, CPU, operation statistics, etc).

The monitoring data will be available on a MongoDB website with a unique URL accessible to you
and anyone you share the URL with. MongoDB may use this information to make product
improvements and to suggest MongoDB products and deployment options to you.

To enable free monitoring, run the following command: db.enableFreeMonitoring()
To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---

> 

```

现在我们再来看下我们配置的数据目录和日志输出信息：

```bash
# 查看数据文件夹下的数据
$ ls /Users/mingrn/Mongodb/data/

WiredTiger				collection-2--2098776402360623939.wt	index-6--2098776402360623939.wt
WiredTiger.lock				collection-4--2098776402360623939.wt	index-7-3467119067361957614.wt
WiredTiger.turtle			collection-6-3467119067361957614.wt	journal
WiredTiger.wt				diagnostic.data				mongod.lock
WiredTigerLAS.wt			index-1--2098776402360623939.wt		sizeStorer.wt
_mdb_catalog.wt				index-3--2098776402360623939.wt		storage.bson
collection-0--2098776402360623939.wt	index-5--2098776402360623939.wt
```

```bash
$ tail -f /Users/mingrn/Mongodb/log/mongodb.log 

# 列出部分日志
2019-07-20T14:56:39.196+0800 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2019-07-20T14:56:39.196+0800 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2019-07-20T14:56:39.196+0800 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
2019-07-20T14:56:39.196+0800 I CONTROL  [initandlisten] 
2019-07-20T14:56:39.196+0800 I CONTROL  [initandlisten] 
2019-07-20T14:56:39.196+0800 I CONTROL  [initandlisten] ** WARNING: soft rlimits too low. Number of files is 256, should be at least 1000
2019-07-20T14:56:39.228+0800 I FTDC     [initandlisten] Initializing full-time diagnostic data capture with directory '/Users/mingrn/Mongodb/data/diagnostic.data'
2019-07-20T14:56:39.231+0800 I NETWORK  [initandlisten] waiting for connections on port 27016
2019-07-20T14:56:40.030+0800 I FTDC     [ftdc] Unclean full-time diagnostic data capture shutdown detected, found interim file, some metrics may have been lost. OK
2019-07-20T15:01:25.069+0800 I NETWORK  [conn1] received client metadata from 127.0.0.1:64595 conn1: { application: { name: "MongoDB Shell" }, driver: { name: "MongoDB Internal Client", version: "4.0.10" }, os: { type: "Darwin", name: "Mac OS X", architecture: "x86_64", version: "18.6.0" } }

```

到这里，就知道该如何自定义配置了。篇幅限制就不一一列举了，只需要知道所有的配置信息使用 `--help` 命令即可查看，再选择需要的参数即可！