# MySQL安装与配置

**这里介绍的通过zip压缩包来安装MySQL**

##### 下载解压MySQL

1. 在[MySQL官网](https://dev.mysql.com/downloads/mysql/)下载安装包：**Windows (x86, 64-bit), ZIP Archive**
2. 在电脑D盘（任何一个盘都可以），新建 `MySQL` 文件夹用来存放解压的MySQL文件。
3.  将下载的压缩包里面的所有文件都解压到新建的 `MySQL` 文件夹中。

##### 配置MySQL文件

1. 进入MySQL文件解压的位置，新建 `data文件夹` 、`新建文本文档.txt` 。

![QQ截图20200309225815](image/QQ截图20200309225815.png)

2. 在 `新建文本文档.txt` 中粘贴下面内容，**注意：修改当中的文件路径配置**。

```
[mysqld]
# 设置3306端口
port=3306
# 设置mysql的安装目录(这里修改为自己的MySQL安装路径)
basedir=D:\Program Files\MySQL
# 设置mysql数据库的数据的存放目录(这里修改为自己的MySQL安装路径加上新建的data文件夹路径)
datadir=D:\Program Files\MySQL\data
# 允许最大连接数
max_connections=200
# 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
max_connect_errors=10
# 服务端使用的字符集默认为UTF8
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 默认使用“mysql_native_password”插件认证
default_authentication_plugin=mysql_native_password

[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8
```

3. 将修改后 `新建文本文档.txt` 文件**另存为编码为 `ANSI`** 后保存。
4. 将文件名称修改为 `my.ini` 后保存， **注意：文件后缀名为 `ini`**。

##### 添加环境变量

1. Win10系统——设置——搜索“环境变量”——点击“环境变量”——系统变量——双击变量名“Path”
2. 点击“新建”——粘贴`D:\Program Files\MySQL\bin ` **注意：这里必须是MySQL下的bin文件夹路径**
3. 点击“确定”——点击“确定”——保存退出

##### 初始化MySQL

1. 进入命令行(管理员模式)，输入下面命令**初始化配置**(报错参考：[找不到 VCRUNTIME140_1.dll](https://blog.csdn.net/qq_42365534/article/details/102847013))：

```
mysqld --initialize-insecure --user=mysql
```

2. 输入下面命令，提示：`Service successfully installed. `就说明配置完成。

```
mysqld -install
```

##### 配置MySQL账户

1. 在进入命令行(管理员模式)，输入下面命令**启动MySQL服务**

```
net start mysql
MySQL 服务正在启动 .
MySQL 服务已经启动成功。
```

2. 输入下面命令**进入MySQL**，因为是第一次进入不需要密码(`-u root`默认账户名为root，`-p`密码登录)：

```
C:\Windows\system32>mysql -u root -p(直接回车)
Enter password:(直接回车)
```

3.出现面下面内容，说明**成功进入到MySQL**：

```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.19 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

4. 在MySQL里面输入下面命令**配置新密码**(这里设置新密码为123456)：

```
mysql> alter user user() identified by "123456";
```

5. 输入下面命令，**退出MySQL**：

```
mysql> quit
Bye
```

6. 输入下面命令，通过**新密码进入MySQL**：

```
C:\Windows\system32>mysql -u root -p(直接回车)
Enter password:******(输入123456)

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.19 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

7. 输入下面命令，退出MySQL，**并结束MySQL服务**：

```
mysql> quit
Bye

C:\Windows\system32>net stop mysql
MySQL 服务正在停止.
MySQL 服务已成功停止。
```