## 源码安装mysql5.7

### 1、清理环境：

```
慎重选择
#yum erase mariadb mariadb-server mariadb-libs mariadb-devel -y
#userdel  -r
#rm  -rf  /etc/my*
#rm  -rf  /var/lib/mysql
```

### 2、创建mysql用户和组

[root@mysql-server ~]# useradd -r mysql -M -s /bin/false

```
[root@node2 ~]# groupadd  mysql
[root@node2 ~]# id myql
id: myql: no such user
[root@node2 ~]# useradd -d /home/myql -g mysql -m mysql
[root@node2 ~]# id mysql
uid=1000(mysql) gid=1000(mysql) 组=1000(mysql)
更改mysql的密码
[root@node2 ~]# passwd mysql
更改用户 mysql 的密码 。
新的 密码：
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。
```

### 3、从官网下载tar包

```
#wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-boost-5.7.27.tar.gz
#wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.27.tar.gz
```

### 4、安装编译工具

```
#yum -y install ncurses ncurses-devel openssl-devel bison gcc gcc-c++ make
#yum -y install  cmake
另一版本
yum install cmake gcc gcc-c++ ncurses-devel bison zlib libxml openssl automake autoconf make libtool bison-devel libaio-devel
```

### 5、创建mysql目录

```
[root@mysql-server ~]# mkdir -p /usr/local/{data,mysql,log}
[root@mysql-server ~]# chown -R mysql:mysql  /usr/local/{data,mysql,log}
```

### 6、解压

```
[root@mysql-server ~]# tar xzvf mysql-boost-5.7.27.tar.gz -C /usr/local/
[root@mysql-server ~]# tar xzvf mysql-5.7.27.tar.gz -C /usr/local/
```

注:如果安装的MySQL5.7及以上的版本，在编译安装之前需要安装boost,因为高版本mysql需要boots库的
安装才可以正常运行。否则会报CMake Error at cmake/boost.cmake:81错误
安装包里面自带boost包

### 7、编译安装

cd 解压的mysql目录

```
[root@mysql-server ~]# cd /usr/local/mysql-5.7.27/
[root@mysql-server mysql-5.7.27]#cmake . \
-DWITH_BOOST=boost/boost_1_59_0/ \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DSYSCONFDIR=/etc \
-DMYSQL_DATADIR=/usr/local/mysql/data \
-DINSTALL_MANDIR=/usr/share/man \
-DMYSQL_TCP_PORT=3306 \
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
-DDEFAULT_CHARSET=utf8 \
-DEXTRA_CHARSETS=all \
-DDEFAULT_COLLATION=utf8_general_ci \
-DWITH_READLINE=1 \
-DWITH_SSL=system \
-DWITH_EMBEDDED_SERVER=1 \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1
```

提示：boost也可以使用如下指令自动下载，如果不下载bost压缩包，把下面的这一条添加到配置中第二行
-DDOWNLOAD_BOOST=1/
参数详解:
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \  安装目录
-DSYSCONFDIR=/etc \  配置文件存放 （默认可以不安装配置文件）
-DMYSQL_DATADIR=/usr/local/mysql/data \  数据目录  错误日志文件也会在这个目录
-DINSTALL_MANDIR=/usr/share/man \   帮助文档
-DMYSQL_TCP_PORT=3306 \   默认端口
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \ sock文件位置，用来做网络通信的，客户端连接服务器的
时候用
-DDEFAULT_CHARSET=utf8 \  默认字符集。字符集的支持，可以调
-DEXTRA_CHARSETS=all \  扩展的字符集支持所有的
-DDEFAULT_COLLATION=utf8_general_ci \ 支持的
-DWITH_READLINE=1 \  上下翻历史命令
-DWITH_SSL=system \  使用私钥和证书登陆（公钥） 可以加密。 适用与长连接。坏处：速度慢
-DWITH_EMBEDDED_SERVER=1 \  嵌入式数据库
-DENABLED_LOCAL_INFILE=1 \  从本地倒入数据，不是备份和恢复。
-DWITH_INNOBASE_STORAGE_ENGINE=1 默认的存储引擎，支持外键

```
[root@mysql-server mysql-5.7.27]# make && make install
```

如果安装出错，想重新安装：
 不用重新解压，只需要删除安装目录中的缓存文件CMakeCache.txt

### 8、初始化

```
[root@mysql-server mysql-5.7.27]# cd /usr/local/mysql
[root@mysql-server mysql]# chown -R mysql.mysql .
[root@mysql-server mysql]# ./bin/mysqld --initialize --user=mysql --
basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
```

   ---初始化完成之后，一
定要记住提示最后的密码用于登陆或者修改密码
![1595835348488](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1595835348488.png)
初始化,只需要初始化一次

```
[root@mysql-server ~]# vim /etc/my.cnf  ---添加如下内容
[mysqld]
basedir=/usr/local/mysql   #指定安装目录
datadir=/usr/local/mysql/data  #指定数据存放目录
```

### 9、启动mysql

```
[root@mysql-server ~]# cd /usr/local/mysql
[root@mysql-server mysql]# ./bin/mysqld_safe --user=mysql &
```

![1595835444499](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1595835444499.png)

### 10、登录mysql

```
[root@mysql-server mysql]# /usr/local/mysql/bin/mysql -uroot -p'GP9TKGgY9i/8'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.27
Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> exit
```

### 11、修改密码

```
[root@mysql-server mysql]# /usr/local/mysql/bin/mysqladmin -u root -
p'GP9TKGgY9i/8' password 'QianFeng@123'
mysqladmin: [Warning] Using a password on the command line interface can be
insecure.
Warning: Since password will be sent to server in plain text, use ssl connection
to ensure password safety.
```

### 12、添加环境变量

```
[root@mysql-server mysql]# vim /etc/profile  ---添加如下
PATH=$PATH:$HOME/bin:/usr/local/mysql/bin
[root@mysql-server mysql]# source /etc/profile
```


之后就可以在任何地方使用mysql命令登陆Mysql服务器：

### 13、配置mysqld服务的管理工具：

```
[root@mysql-server mysql]# cd /usr/local/mysql/support-files/
[root@mysql-server support-files]# cp mysql.server /etc/init.d/mysqld
[root@mysql-server support-files]# chkconfig --add mysqld
[root@mysql-server support-files]# chkconfig mysqld on
```

先将原来的进程杀掉

```
[root@mysql-server ~]# /etc/init.d/mysqld start
Starting MySQL. SUCCESS!
[root@mysql-server ~]# netstat -lntp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address      Foreign Address     State   
PID/Program name  
tcp     0    0 0.0.0.0:22        0.0.0.0:*        LISTEN  
 1087/sshd     
tcp6    0    0 :::22          :::*          LISTEN  
 1087/sshd     
tcp6    0    0 :::3306         :::*          LISTEN  
 31249/mysqld
[root@mysql-server ~]# /etc/init.d/mysqld stop
```

数据库编译安装完成







