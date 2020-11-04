# MySQL安装与配置

MySQL安装文件大体上有两种，一种是**zip压缩包**，一种是**msi文件**。

**zip**：一种数据压缩和文档储存的文件格式，属于几种主流的压缩格式之一。

**msi文件**：Windows Installer的数据包，它实际上是一个数据库，包含安装一种产品所需要的信息和在很多安装情形下安装（和卸载）程序所需的指令和数据。

### zip压缩包安装

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

### mis文件安装

##### mis文件安装过程

![QQ截图20200321144628](image/QQ截图20200321144628.png)

双击安装包

![QQ截图20200321144643](image/QQ截图20200321144643.png)

准备安装，等待几分钟

![QQ截图20200321144704](image/QQ截图20200321144704.png)

选中 I accept...，再点击Next

![QQ截图20200321144719](image/QQ截图20200321144719.png)

选中最下方的Custom，再点击Next

![QQ截图20200321144739](image/QQ截图20200321144739.png)

依次点击MySQL Servers——MySQL  Server——MySQL  Server 5.7——MySQL  Server 5.7.17 - X64，点击中间向右箭头

![QQ截图20200321144752](image/QQ截图20200321144752.png)

仅选中MySQL Server、Client Programs，点击Next

![QQ截图20200321144805](image/QQ截图20200321144805.png)

点击Execute，等待几分钟

![QQ截图20200321144820](image/QQ截图20200321144820.png)

继续等待

![QQ截图20200321144838](image/QQ截图20200321144838.png)

出现如图绿色小对号说明安装成功，点击Next

![QQ截图20200321144855](image/QQ截图20200321144855.png)

确保Config Type中是Development Machine，点击Next

![QQ截图20200321144909](image/QQ截图20200321144909.png)

设置MySQL的密码（需要牢记），点击Next

![QQ截图20200321144923](image/QQ截图20200321144923.png)

Windows Service Name选项为MySQL实例名称，无需更改

Start the MySQL Server at System Startup选项为开机自启，可以不勾选

![QQ截图20200321144934](image/QQ截图20200321144934.png)

如图所示就可以点击Next

![QQ截图20200321144951](image/QQ截图20200321144951.png)

点击Execute，等待几分钟

![QQ截图20200321145004](image/QQ截图20200321145004.png)

点击Finish

![QQ截图20200321145019](image/QQ截图20200321145019.png)

点击Next

![QQ截图20200321145030](image/QQ截图20200321145030.png)

点击Finish

##### 配置环境变量

![QQ截图20200321145042](image/QQ截图20200321145042.png)

复制MySQL的安装路径，配置环境变量时需要

![QQ截图20200321145106](image/QQ截图20200321145106.png)

依次点击“我的电脑”——右键“属性”——”高级系统设置”

![QQ截图20200321145118](image/QQ截图20200321145118.png)

点击“环境变量”

![QQ截图20200321145131](image/QQ截图20200321145131.png)

选中“系统变量”中“Path”，点击“编辑”

![QQ截图20200321145146](image/QQ截图20200321145146.png)

点击“新建”

![QQ截图20200321145158](image/QQ截图20200321145158.png)

再下方添加拷贝的MySQL安装路径，点击“确定”

![QQ截图20200321145207](image/QQ截图20200321145207.png)

点击“确定”

![QQ截图20200321145217](image/QQ截图20200321145217.png)

点击“确定”

##### MySQL终端

以“管理员模式”启动命令行，输入 `net start mysql实例的名字`，回车启动MySQL服务

![QQ截图20200321145302](image/QQ截图20200321145302.png)

输入 `mysql -u root -p` 回车，在输入安装MySQL时配置的密码，回车，进入MySQL

![QQ截图20200321145316](image/QQ截图20200321145316.png)