# 数据结构、内部编码

**Redis有5种数据结构分别是： string（字符串）、hash（哈希）、list（列表）、set（集合）、zset（有序集合）。**

![QQ截图20200517153651](image/QQ截图20200517153651.png)

### 字符串

**字符串类型是Redis最基础的数据结构。**

**首先键都是字符串类型，而且其他几种数据结构都是在字符串类型基础上构建的。**

字符串类型的值实际可以是字符串（简单的字符串、复杂的字符串（例如JSON、XML））、数字 （整数、浮点数），甚至是二进制（图片、音频、视频），但是**值最大不能超过512MB**。

![QQ截图20200517154010](image/QQ截图20200517154010.png)

##### 添加值、获取值

**添加值**：添加一个键值对。

```
set key value [ex seconds] [px milliseconds] [nx|xx]
```

- ex seconds：为键设置秒级过期时间。 

- px milliseconds：为键设置毫秒级过期时间。 

- nx：**键必须不存在，才可以设置成功，用于添加。** 

- xx：**与nx相反，键必须存在，才可以设置成功，用于更新。**

上述的命令列举的参数，都可以和`set` 合并：

```
set key value ex seconds == setex key seconds value
set key value px seconds == setpx key seconds value
set key value nx == setnx key value
set key value xx == setxx key value
```

**获取值**：获取一个键值对的值。

```
get key
```

设置键为hello，值为world的键值对，**返回结果为OK代表设置成功**：

```
127.0.0.1:6379> set hello world 
OK
```

获取键hello的值，**如果要获取的键不存在，则返回nil（空）**：

```
127.0.0.1:6379> get hello 
"world"

127.0.0.1:6379> get not_exist_key 
(nil)
```

##### 批量添加、批量获取

批量添加：批量添加键值对。

```
mset key value [key value ...]
```

批量获取命令：批量获取键值。

```
mget key [key ...]
```

通过mset命令一次性设置4个键值对：

```
127.0.0.1:6379> mset a 1 b 2 c 3 d 4 
OK
```

批量获取了键a、b、c、d、f的值，**f键不存在，那么它的值为nil（空）**：

```
127.0.0.1:6379> mget a b c d f
1) "1" 
2) "2" 
3) "3" 
4) "4"
5) (nil)
```

批量操作命令可以有效提高开发效率，假如没有批量操作命令，执行n次get命令需要按下面方式来执行：

![QQ截图20200517160112](image/QQ截图20200517160112.png)

使用批量操作命令后，要执行n次get命令操作只需要按照下面方式来完成：

![QQ截图20200517160213](image/QQ截图20200517160213.png)

假设网络时间为1毫秒，命令时间为0.1毫秒（按照每秒处理1万条命令算），那么执行1000次get命令和1次mget命令的区别如下：

![QQ截图20200517160348](image/QQ截图20200517160348.png)

##### 长度、切片、替换

**长度**：获取键值长度。

```
strlen key
```

例如，当前值为redisworld，所以返回值为10：

```
127.0.0.1:6379> get key 
"redisworld" 
127.0.0.1:6379> strlen key 
(integer) 10
```

!> 每个中文占用3个字节长度。

**切片：**`start` 和 `end` 分别是开始位置和结束位置，开始位置从0开始计算，**前闭后闭区间**。

```
getrange key start end
```

取best的前两个字符：

```
127.0.0.1:6379> getrange redis 0 1 
"be"
```

**替换**：替换指定位置的字符。

```
setrange key offeset value
```

下面操作将值由pest变为了best：

```
127.0.0.1:6379> set redis pest 
OK
127.0.0.1:6379> setrange redis 0 b 
(integer) 4 
127.0.0.1:6379> get redis 
"best"
```

##### 追加值、返回原值

追加值： **`append` 可以向字符串尾部追加值**。

```
append key value
```

返回原值：**`getset` 设置值的同时会返回键原来的值**。

```
getset key value
```

字符串后面追加字符串：

```
127.0.0.1:6379> set key redis
OK
127.0.0.1:6379> get key
"redis"
127.0.0.1:6379> append key world
(integer) 10
127.0.0.1:6379> get key
"redisworld"
```

获取原来字符串：

```
127.0.0.1:6379> getset hello world 
(nil) 
127.0.0.1:6379> getset hello redis 
"world"
```

##### 计数

```
incr key
```

`incr`命令用于对值做自增操作，返回结果分为三种情况： 

- **值不是整数，返回错误。**
- **值是整数，返回自增后的结果**。
- **键不存在，按照值为0自增，返回结果为1。** 

例如**对一个不存在的键执行incr操作后，返回结果是1，再次对键执行incr命令，返回结果是2**：

```
127.0.0.1:6379> exists key 
(integer) 0 
127.0.0.1:6379> incr key 
(integer) 1
127.0.0.1:6379> incr key 
(integer) 2
```

如果值不是整数，那么会返回错误：

```
127.0.0.1:6379> set hello world 
OK
127.0.0.1:6379> incr hello 
(error) ERR value is not an integer or out of range
```

除了incr命令，Redis提供了`decr`（自减）、 `decrby`（自减指定数字）、`incrby`（自增指定数字）、`incrbyfloat`（自增浮点数）：

```
decr key 
decrby key decrement 
incrby key increment 
incrbyfloat key increment
```

### 哈希

**在Redis中，哈希类型是指键值本身又是一个键值对结构**，形如`value={{field1，value1}，...{fieldN，valueN}}`。

![QQ截图20200519224409](image/QQ截图20200519224409.png)

!> 哈希类型中的映射关系叫作 `field-value`，注意这里的 `value` 是指 `field` 对应的值，不是键对应的值。

##### 设置值

```
hset key field value
```

为 `user` 添加一对 `field-value`，**如果设置成功会返回1，反之会返回0**。

```
127.0.0.1:6379> hset user name tom 
(integer) 1
```

##### 获取值

```
hget key field
```

获取 `user` 的 `name` 域（属性）对应的值：

```
127.0.0.1:6379> hget user name 
"tom"
```

**如果键或 `field` 不存在，会返回 `nil`**

```
127.0.0.1:6379> hget user:1 age 
(nil)

127.0.0.1:6379> hget user:2 name 
(nil) 
```











### 内部编码

**Redis对外有5种数据结构分别是： string（字符串）、hash（哈希）、list（列表）、set（集合）、zset（有序集合），实际上每种数据结构都有自己底层的内部编码实现，而且是多种实现**，这样Redis会在合适的场景选择合适的内部编码。

![QQ截图20200517152640](image/QQ截图20200517152640.png)

##### 查询内部编码

可以看到每种数据结构都有两种以上的内部编码实现，可以通过 `object encoding` 命令查询内部编码：

```
127.0.0.1:6379> set hello world 
OK
127.0.0.1:6379> object encoding hello 
"embstr" 

127.0.0.1:6379> rpush mylist a b c d e f g 
(integer) 7
127.0.0.1:6379> object encoding mylist 
"ziplist"
```

Redis这样设计有两个好处：第一，可以改进内部编码，而对外的数据结构和命令没有影响，这样一旦开发出更优秀的内部编码，无需改动外部数据结构和命令。第二，多种内部编码实现可以在不同场景下发挥各自的优势。

##### 字符串编码

字符串类型的内部编码有3种： 

- int：8个字节的长整型。 

- embstr：小于等于39个字节的字符串。 

- raw：大于39个字节的字符串。 

Redis会根据当前值的类型和长度决定使用哪种内部编码实现。

整数类型示例如下：

```
127.0.0.1:6379> set key 8653 
OK
127.0.0.1:6379> object encoding key 
"int"
```

短字符串示例如下：

```
#小于等于39个字节的字符串：embstr 
127.0.0.1:6379> set key "hello,world" 
OK
127.0.0.1:6379> object encoding key 
"embstr"
```

长字符串示例如下：

```
#大于39个字节的字符串：raw 
127.0.0.1:6379> set key "one string greater than 39 byte........." 
OK
127.0.0.1:6379> object encoding key 
"raw" 
127.0.0.1:6379> strlen key 
(integer) 40
```