# 连接、查看与选择、退出

### 连接

##### 启动MySQL服务

在命令行(管理员模式)中，输入下面命令，启动MySQL服务：

```
net start mysql的实例名称
```

MySQL的实例名称和版本有关，例如MySQL5.7实例名称有可能时mysql57，一般情况下mysql的实例名称就是mysql。

##### 连接MySQL

MySQL的连接密码在前面[MySQL01-MySQL安装与配置](/MySQL01-MySQL安装与配置.md)已经设置，这里不再赘述。

```
# 连接服务器的MySQL
mysql -h 服务器地址 -P 端口号 -u 用户名 -p（回车）
Enter Password:（输入密码）
mysql>（进入成功）

# 连接本机的MySQL，第一次进入本机的用户名默认为root
mysql -u root -p（回车）
Enter Password:（输入密码）
mysql>（进入成功）
```

进入MySQL界面如下：

```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 38
Server version: 8.0.19 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

### 查看与选择

##### 查看数据版本

```
方式一：登录到MySQL服务器
SELECT VERSION();
方式二：没有登录到MySQL服务器
mysql --version 或 mysql --V
```

##### 查看全部数据库

查看当前所有的数据库：`SHOW DATABASES;`

```sql
mysql> SHOW DATABASES;

/*
列出当前所有的数据库，共5行
+--------------------+
| Database           |
+--------------------+
| information_schema |
| lb                 |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
*/
```

##### 选择数据库

选中某一个数据库：`USE 数据库名称;`

!> 注意：**被选择的数据一定要是存在的。**

```sql
-- 选择存在的数据库
mysql> USE lb;
/*
数据库已被选择
Database changed
*/

-- 选择不存在的数据
mysql> USE 123;
/*
报错：选择未知数据库
ERROR 1049 (42000): Unknown database '123'
*/
```

##### 查看当前位置

查看当前所在的位置：`SELECT DATABASE();`

```sql
-- 未选择数据库
mysql> SELECT DATABASE();
/*
未选择数据库，返回位置信息为NULL
+------------+
| DATABASE() |
+------------+
| NULL       |
+------------+
1 row in set (0.00 sec)
*/

-- 选择数据库
mysql> USE lb;
/*
Database changed
*/

-- 再次查看位置
mysql> SELECT DATABASE();
/*
返回当前所在数据库的位置信息
+------------+
| DATABASE() |
+------------+
| lb         |
+------------+
1 row in set (0.00 sec)
*/
```

##### 查看库中所有表

查看数据库中所有的表：`SHOW TABLES;` 或者 `SHOW TABLES FROM 数据库名;`

!> 注意：**使用 `SHOW TABLES;` 查看表之前一定要选中存在的数据库。**

```sql
-- 已经选择了存在的数据库
mysql> SHOW TABLES;
/*
列出当前数据库中所有的表，共1行
+--------------------+
| Tables_in_lb       |
+--------------------+
| 通用认证信息       |
+--------------------+
1 row in set (0.00 sec)
*/

-- 未选择数据库
mysql> SHOW TABLES;
/*
报错，没有数据库被选中
ERROR 1046 (3D000): No database selected
*/

-- 通过语句语句选择数据库查看表（但是当前所在位置不变，并没有进入数据库）
mysql> SHOW TABLES FROM lb;
/*
+--------------------+
| Tables_in_lb       |
+--------------------+
| 通用认证信息       |
+--------------------+
1 row in set (0.00 sec)
*/
```

##### 查看表结构

查看一张表的字段结构：`SHOW COLUMNS FROM 表名;` 或 `DESCRIBE 表名;` 或 `DESC 表名;`

注意：**查看表结构之前一定要选中存在的数据库，而且表已经是存在的。**

```sql
mysql> SHOW COLUMNS FROM 通用认证信息;
/*
返回表结构信息，共7行
+--------------------+---------------+------+-----+---------+----------------+
| Field              | Type          | Null | Key | Default | Extra          |
+--------------------+---------------+------+-----+---------+----------------+
| ID                 | int           | NO   | PRI | NULL    | auto_increment |
| 公司名称           | varchar(1000) | YES  |     | NULL    |                |
| 证书编号           | varchar(1000) | YES  |     | NULL    |                |
| 认证项目           | varchar(1000) | YES  |     | NULL    |                |
| 证书状态           | varchar(1000) | YES  |     | NULL    |                |
| 证书到期日期       | varchar(1000) | YES  |     | NULL    |                |
| 发证机构           | varchar(1000) | YES  |     | NULL    |                |
+--------------------+---------------+------+-----+---------+----------------+
7 rows in set (0.00 sec)

# Field列：表中的每一个字段的名称。
# Type列：当前字段存储的数据类型，int整形，varchar(1000)长度为1000的字符型。
# Null列：NO当前字段不能为空，YES当前字段可以为空。
# Key列：PRI主键字段。
# Default列：NULL当前字段默认值为空。
# Extra列：auto_increment当前字段有自增属性，只针对于int类型字段。
*/
```

##### 其他查看命令

```
SHOW STATUS，用于显示广泛的服务器状态信息；
SHOW CREATE DATABASE和SHOW CREATE TABLE，分别用来显示创建特定数据库或表的MySQL语句；
SHOW GRANTS，用来显示授予用户（所有用户或特定用户）的安全权限；
SHOW ERRORS和SHOW WARNINGS，用来显示服务器错误或警告消息。
```

### 退出

退出MySQL命令：`exit` 或 `quit`

注意：**退出命令后面可以没有 `;`**。

```sql
mysql> exit
/*
成功退出MySQL
Bye
*/
```