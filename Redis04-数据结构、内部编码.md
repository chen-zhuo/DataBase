# 数据结构、内部编码

**Redis有5种数据结构分别是： string（字符串）、hash（哈希）、list（列表）、set（集合）、zset（有序集合）。**

![QQ截图20200517153651](image/QQ截图20200517153651.png)

### 字符串

**字符串类型是Redis最基础的数据结构，其他几种数据结构都是在字符串类型基础上构建的。**

字符串类型的值实际可以是字符串（简单的字符串、复杂的字符串（例如JSON、XML））、数字 （整数、浮点数），甚至是二进制（图片、音频、视频），但是**值最大不能超过512MB**。

![QQ截图20200517154010](image/QQ截图20200517154010.png)

##### 添加、获取

添加：**添加一个键值对**。

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

获取：**获取一个键值对的值**。

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

批量添加：**批量添加键值对**。

```
mset key value [key value ...]
```

批量获取：**批量获取键值**。

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

!> 在Redis中每个汉字的长度为3。

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

### 列表

**列表（`list`）类型是用来存储多个有序的字符串。列表中的每个字符串称为元素（`element`），一个列表最多可以存储232-1个元素。**在Redis中，可以对列表两端插入（`push`）和弹出（`pop`），还可以获取指定范围的元素列表、获取指定索引下标的元素等。

列表类型有两个特点：

- **列表中的元素是有序的，这就意味着可以通过索引下标获取某个元素或者某个范围内的元素列表**。
- **列表中的元素可以是重复的**。

**列表是一种比较灵活的数据结构，它可以充当栈和队列的角色，在实际开发上有很多应用场景。**

![QQ截图20200520230103](image/QQ截图20200520230103.png)

![QQ截图20200520230332](image/QQ截图20200520230332.png)

##### 添加、查找、删除

右添加：**从右边插入一个或多个元素。**

``` 
rpush key value [value ...]
```

左添加：**从左边插入一个或多个元素。**

```
lpush key value [value ...]
```

位置添加：**`linsert` 命令会从列表中找到等于 `pivot` 的元素，在其前（before）或者后（after）插入一个新的元素 `value`。**

```
linsert key before|after pivot value
```

范围查找：**获取指定范围内的元素列表。**

`lrange` 命令会获取列表指定索引范围所有的元素。索引下标有两个特点：

- **索引下标从左到右分别是0到N-1，但是从右到左分别是-1到-N。**
- **`lrange`中的end选项包含了自身，这个和很多编程语言不包含end不太相同。**

```
lrange key start end
```

位置查找：**获取列表指定索引下标的元素。**

```
lindex key index
```

左删除：**从列表左侧弹出元素。**

```
lpop key
```

右删除：**从列表右侧弹出。** 

```
rpop key
```

指定删除：**删除指定元素。**

`lrem`命令会从列表中找到等于 `value` 的元素进行删除，根据count的不同分为三种情况： 

- **count>0，从左到右，删除最多count个元素。**
- **count<0，从右到左，删除最多count绝对值个元素。**
- **count=0，删除所有。**

```
lrem key count value
```

在列表最右边依次插入元素c、b、a：

```
127.0.0. 1:6379> rpush listkey c b a 
(integer) 3
```

在列表的元素b前插入 java：

```
127.0.0.1:6379> linsert listkey before b java 
(integer) 4
```

获取列表的第2到第4个元素：

```
127.0.0.1:6379> lrange listkey 1 3 
1) "java" 
2) "b" 
3) "a"
```

获取列表最后一个元素：

```
127.0.0.1:6379> lindex listkey -1 
"a"
```

将列表最左侧的元素c会被弹出，弹出后列表变为java、b、a：

```
127.0.0.1:6379>t lpop listkey 
"c" 

127.0.0.1:6379> lrange listkey 0 -1 
1) "java" 
2) "b" 
3) "a"
```

向列表从左向右插入5个a，那么当前列表变为“a a a a a java b a”，下面操作将从列表左边开始删除4个为a的元素：

```
127.0.0.1:6379> lrem listkey 4 a 
(integer) 4 
127.0.0.1:6379> lrange listkey 0 -1 
1) "a" 
2) "java" 
3) "b" 
4) "a"
```

##### 长度、修剪

长度：**计算列表长度。**

```
llen key
```

修剪：**按照索引范围修剪列表。**

```
ltrim key start end
```

下面示例当前列表长度为4：

```
127.0.0.1:6379> llen listkey 
(integer) 4
```

保留列表listkey第2个到第4个元素：

```
127.0.0.1:6379> ltrim listkey 1 3 
OK

127.0.0.1:6379> lrange listkey 0 -1 
1) "java" 
2) "b" 
3) "a"
```

### 哈希

**在Redis中，哈希类型是指键值本身又是一个键值对结构**，形如`value={{field1，value1}，...{fieldN，valueN}}`。

![QQ截图20200519224409](image/QQ截图20200519224409.png)

!> 哈希类型中的映射关系叫作 `field-value`，注意这里的 `value` 是指 `field` 对应的值，不是键对应的值。

##### 添加、获取、删除

添加：**添加一对域值**。

```
hset key field value
```

获取：**获取域的属性值**。

```
hget key field
```

删除：删除除一个或多个 `field`，**返回结果为成功删除 `field` 的个数**。

```
hdel key field [field ...]
```

为 `user` 添加一对 `field-value`，**如果设置成功会返回1，反之会返回0**。

```
127.0.0.1:6379> hset user name tom 
(integer) 1
```

获取 `user` 的 `name` 域（属性）对应的值，**如果键或 `field` 不存在，会返回 `nil`**：

```
127.0.0.1:6379> hget user name 
"tom"

127.0.0.1:6379> hget user age 
(nil)

127.0.0.1:6379> hget user2 name 
(nil) 
```

删除 `user` 的 `name` 域，**如果域不存在返回0**：

```
127.0.0.1:6379> hdel user name
(integer) 1 

127.0.0.1:6379> hdel user name
(integer) 0
```

##### 批量添加、批量获取

`hmset` 和 `hmget` 分别是批量设置和获取 `field-value`，`hmset` 需要的参数是 `key` 和多对 `field-value`，`hmget` 需要的参数是 `key` 和多个 `field`。

```
hmget key field [field ...] 
hmset key field value [field value ...]
```

批量添加、批量获取多个域值：

```
127.0.0.1:6379> hmset user name mike age 12 city tianjin 
OK
127.0.0.1:6379> hmget user name city 
1) "mike" 
2) "tianjin"
```

获取全域、获取全值

获取所有 `field`：

```
hkeys key
```

获取所有的`value`：

```
hvals key
```

获取所有的 `field-value`：

```
hgetall key
```

获取 `user` 全部 `field` 和 `value` 和 `field-value`：

```
127.0.0.1:6379> hkeys user 
1) "name" 
2) "age" 
3) "city"

127.0.0.1:6379> hvals user 
1) "mike" 
2) "12" 
3) "tianjin"

127.0.0.1:6379> hgetall user 
1) "name" 
2) "mike" 
3) "age" 
4) "12" 
5) "city" 
6) "tianjin"
```

?> 在使用 `hgetall` 时，如果哈希元素个数比较多，会存在阻塞Redis的可能。如果开发人员只需要获取部分 `field`，可以使用 `hmget`，如果一定要获取全部 `field-value`，可以使用 `hscan` 命令，该命令会渐进式遍历哈希类型。

##### 判断、统计、计算

判断 `field` 是否存在，存在返回结果为1，不存在时返回0：

```
hexists key field
```

统计 `field` 个数:

```
hlen key
```

计算 `value` 的字符串长度：

```
hstrlen key field
```

例如 `user` 有3个 `field`：

```
127.0.0.1:6379> hset user name tom 
(integer) 1 
127.0.0.1:6379> hset user age 23 
(integer) 1 
127.0.0.1:6379> hset user city tianjin 
(integer) 1 

127.0.0.1:6379> hexists user name 
(integer) 1

127.0.0.1:6379> hlen user 
(integer) 3

127.0.0.1:6379> hstrlen user name 
(integer) 3
```

##### 优势劣势

用户信息存储更直观：

![QQ截图20200520225028](image/QQ截图20200520225028.png)

哈希类型是稀疏的，而关系型数据库是完全结构化的，例如哈希类型每个键可以有不同的`field`，而关系型数据库一旦添加新的列，所有行都要为其设置值（即使为NULL）。

关系型数据库可以做复杂的关系查询，而Redis去模拟关系型复杂查询开发困难，维护成本高。 

![QQ截图20200520225340](image/QQ截图20200520225340.png)





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

##### 哈希编码

哈希类型的内部编码有两种：

- `ziplist`（压缩列表）：当哈希类型元素个数小于 `hash-max-ziplist-entries` 配置（默认512个）、同时所有值都小于 `hash-max-ziplist-value` 配置（默认64 字节）时，Redis会使用 `ziplist` 作为哈希的内部实现，`ziplist` 使用更加紧凑的结构实现多个元素的连续存储，所以在节省内存方面比 `hashtable` 更加优秀。
- `hashtable`（哈希表）：当哈希类型无法满足 `ziplist` 的条件时，Redis会使用 `hashtable` 作为哈希的内部实现，因为此时 `ziplist` 的读写效率会下降，而 `hashtable` 的读写时间复杂度为 `O(1)`。

当 `field` 个数比较少且没有大的 `value` 时，内部编码为 `ziplist`： 

```
127.0.0.1:6379> hmset hashkey f1 v1 f2 v2 
OK

127.0.0.1:6379> object encoding hashkey 
"ziplist" 
```

当有 `value` 大于64字节，内部编码会由 `ziplist` 变为 `hashtable`： 

```
127.0.0.1:6379> hset hashkey f3 "one string is bigger than 64 byte...忽略..." 
OK

127.0.0.1:6379> object encoding hashkey 
"hashtable" 
```

当 `field` 个数超过512，内部编码也会由 `ziplist` 变为 `hashtable`： 

```
127.0.0.1:6379> hmset hashkey f1 v1 f2 v2 f3 v3 ...忽略... f513 v513
OK

127.0.0.1:6379> object encoding hashkey 
"hashtable" 
```