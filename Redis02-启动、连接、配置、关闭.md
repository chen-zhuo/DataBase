# 文件、启动、连接、关闭

### 文件

Redis的安装文件如下：

![QQ截图20200517020314](image/QQ截图20200517020314.png)

Redis开头可执行文件，称之为Redis Shell，这些可执行文件可以做很多事情，例如：启动和停止Redis、检测和修复Redis的持久化文件，还可以检测Redis的性能。

![QQ截图20200517020627](image/QQ截图20200517020627.png)

### 启动

有三种方法启动Redis：默认配置、运行配置、配置文件。

##### 默认配置

在命令行输入下面命令启动Redis：

```
redis-server
```

![QQ截图20200517020951](image/QQ截图20200517020951.png)

可以看到直接使用redis-server启动Redis后，会打印出一些日志，通过日志可以看到一些信息：

1. 当前的Redis版本的是3.2.100
2. Redis的默认端口是6379
3. 进程ID是33644

!> 因为直接启动无法自定义配置，所以这种方式是不会在生产环境中使用的。

##### 运行启动

redis-server加上要修改配置名和值（可以是多对），没有设置的配置将使用默认配置。

例如，如果要用6380作为端口启动Redis，那么可以执行：

```
redis-server --port 6380
```

![QQ截图20200517022132](image/QQ截图20200517022132.png)

!> 虽然运行配置可以自定义配置，但是如果需要修改的配置较多或者希望将配置保存到文件中，不建议使用这种方式。

##### 配置文件

第一种启动方式的界面有一行提示：`没有读配置文件redis.conf`

```
# Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
```

![20190119141418819](image/20190119141418819.png)

**Redis的配置文件 `redis.conf` 是在Linux系统下名称，在Windows系统下配置文件名称就是 `redis.windows.conf`。**

![QQ截图20200517023357](image/QQ截图20200517023357.png)

将配置写到指定文件里，例如我们将配置写到了 `redis.windows.conf` 中，那么只需要执行如下命令即可启动Redis：

```
redis-server "路径\redis.windows.conf"
```

![20190119140700561](image/20190119140700561.png)

Redis有60多个配置，这里只给出一些重要的配置：

![QQ截图20200517022440](image/QQ截图20200517022440.png)

?> 假如一台机器上启动多个Redis，通过配置文件启动的方式提供了更大的灵活性，所以大部分生产环境会使用这种方式启动Redis。

### 连接

##### 交互式方式

现在我们已经启动了Redis服务，**后面要确保Redis服务的窗口一直处于运行状态（不关闭状态），否则后面的操作将无效**，接下来使用redis-cli连接Redis服务器。

**在Redis服务的窗口处于运行状态下，再启动一个命令行**，输入下面命令连接Redis服务器：

```
redis-cli -h 127.0.0.1 -p 6379 -a password
```

- -h参数：主机地址，默认连接127.0.0.1；

- -p参数：Redis端口，默认6379；
- -a参数：如果Redis配置了密码，就要用这个选项进行密码登录；

**如果-h和-p和-a都没写就是连接 `127.0.0.1:6379` 这个没有密码的Redis实例。**

![QQ截图20200517025015](image/QQ截图20200517025015.png)

##### 通信测试

 PING命令：通常用于测试与服务器的连接是否仍然生效，或者用于测量延迟值。

使用客户端向 Redis 服务器发送一个 PING ，如果服务器运作正常的话，会返回PONG ，否则返回一个连接错误。

```
127.0.0.1:6379> ping
PONG
```

### 关闭

Redis提供了shutdown命令来停止Redis服务。

停止Redis服务：

```
redis-cli shutdown
```

停止Redis服务前，生成持久化文件：

```
redis-cli shutdown nosave|save
```

停止Redis服务后，当使用redis-cli再次连接该Redis服务时，看到Redis已经“失联”。

```
C:\Users\ChenZhuo>redis-cli
Could not connect to Redis at 127.0.0.1:6379: Connection refused
```

?> Redis关闭的过程：断开与客户端的连接、持久化文件生成，是一种相对优雅的关闭方式。

?> 除了可以通过shutdown命令关闭Redis服务以外，还可以通过kill进程号的方式关闭掉Redis。

!>不要粗暴地使用kill-9强制杀死Redis服务，不但不会做持久化操作，还会造成缓冲区等资源不能被优雅关闭，极端情况会造成AOF和复制丢失数据的情况。

