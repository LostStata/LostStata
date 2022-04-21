mysql8.0的安装

官网下载mysql：https://dev.mysql.com/downloads/mysql/

![image-20220420150700297](https://cdn.jsdelivr.net/gh/LostStata/pictures@main/Mysql/202204201507447.png)

下载解压包

```
[root@bogon ~]# wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.28-linux-glibc2.12-x86_64.tar.xz
[root@bogon ~]# tar -xf mysql-8.0.28-linux-glibc2.12-i686.tar.xz -C /usr/local/
[root@bogon ~]# cd /usr/local/
[root@bogon local]# mv mysql-8.0.28-linux-glibc2.12-i686/ mysql-8.0.28
```

清理环境

``` shell
[root@bogon local]# rpm -aq mariadb*
mariadb-libs-5.5.56-2.el7.x86_64
[root@bogon local]# yum remove -y `rpm -aq mariadb*`
```

创建mysql用户

``` shell
[root@bogon local]# groupadd mysql
[root@bogon local]# useradd -r -g mysql mysql

```

 创建mysqldata目录

```shell
[root@bogon local]# mkdir /usr/local/mysql-8.0.28/data
[root@bogon local]# chown mysql:mysql /usr/local/mysql-8.0.28/
[root@bogon local]# chmod 750 /usr/local/mysql-8.0.28/
```

配置my.cnf

``` shell
[root@bogon mysql-8.0.28]# vi /etc/my.cnf
[root@bogon mysql-8.0.28]# cat /etc/my.cnf 
[mysqld]
basedir=/usr/local/mysql-8.0.28
datadir=/usr/local/mysql-8.0.28/data
port=3306
socket=/tmp/mysql.sock
character_set_server=utf8
lower_case_table_names=1
log-error=/usr/local/mysql-8.0.28/data/mysql.log
pid-file=/usr/local/mysql-8.0.28/data/mysql.pid
[mysql]
default-character-set = utf8
```



``` shell
[mysqld]
basedir=/usr/local/mysql-8.0.28
datadir=/usr/local/mysql-8.0.28/data
port=3306
socket=/tmp/mysql.sock
character_set_server=utf8
lower_case_table_names=1
log-error=/usr/local/mysql-8.0.28/data/mysql.log
pid-file=/usr/local/mysql-8.0.28/data/mysql.pid
[mysql]
default-character-set = utf8

```



初始化mysql

``` shell
[root@bogon mysql-8.0.28]# ./bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql-8.0.28/ --datadir=/usr/local/mysql-8.0.28/data/-bash: ./bin/mysqld: /lib/ld-linux.so.2: bad ELF interpreter: No such file or directory
出现报错，安装yum -y install glibc.i686
安装完成后，继续报错

[root@bogon mysql-8.0.28]# ./bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql-8.0.28/ --datadir=/usr/local/mysql-8.0.28/data/./bin/mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory
安装yum -y install zlib.i686 --setopt=protected_multilib=false <!--setpot参数处理多个库共存冲突-->，依然报错

[root@bogon mysql-8.0.28]# ./bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql-8.0.28/ --datadir=/usr/local/mysql-8.0.28/data/./bin/mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory

安装
[root@bogon mysql-8.0.28]# yum install -y libaio.so.1
发现还是报错

继续安装libnuma.so.1
[root@bogon mysql-8.0.28]# yum -y install libnuma.so.1 --setopt=protected_multilib=false

新报错：./bin/mysqld: error while loading shared libraries: libstdc++.so.6: cannot open shared object file: No such file or directory

安装缺失的软件
[root@bogon mysql-8.0.28]# yum whatprovides libstdc++.so.6
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
libstdc++-4.8.5-44.el7.i686 : GNU Standard C++ Library
Repo        : base
Matched from:
Provides    : libstdc++.so.6
[root@bogon mysql-8.0.28]# yum -y install libstdc++-4.8.5-44.el7.i686 --setopt=protected_multilib=false

安装反复报错，最后找到安装这个

此时继续执行mysqld --initialize还是包缺少i686组件错误，博主后来不断安装各种缺少的组件，发来发现这样子不是办法，估计后面缺少上百个这样子的组件，这样子一个个安装不知道什么时候才能结束，后来我继续到网上查找方法，终于找到一个安装所有32位程序需要组件的方法：
安装yum -y install xulrunner.i686
[root@bogon mysql-8.0.28]# ./bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql-8.0.28/ --datadir=/usr/local/mysql-8.0.28/data/ 

[root@bogon mysql-8.0.28]# 
执行成功，在这个位置查看密码：/usr/local/mysql-8.0.28/data/mysql.log
[root@bogon mysql-8.0.28]# cat /usr/local/mysql-8.0.28/data/mysql.log
2022-04-20T08:39:29.003424Z 0 [System] [MY-013169] [Server] /usr/local/mysql-8.0.28/bin/mysqld (mysqld 8.0.28) initializing of server in progress as process 19622
2022-04-20T08:39:29.005799Z 0 [Warning] [MY-013242] [Server] --character-set-server: 'utf8' is currently an alias for the character set UTF8MB3, but will be an alias for UTF8MB4 in a future release. Please consider using UTF8MB4 in order to be unambiguous.
2022-04-20T08:39:29.020335Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2022-04-20T08:39:30.090854Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2022-04-20T08:39:33.697798Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: l3XMt.f4q>Ya

```

配置mysql环境变量

```shell
vi /etc/profile
添加如下
MYSQL_HOME="/usr/local/mysql-8.0.28"
PATH="$PATH:$MYSQL_HOME/bin"
[root@bogon mysql-8.0.28]# source /etc/profile

```

配置mysql的server

``` shell
[root@bogon ~]# cd /usr/local/mysql-8.0.28/support-files/
[root@bogon support-files]# cp mysql.server /etc/init.d/mysqld
使用service mysql start开启
使用service mysql stop关闭
配置mysql的开启自启
[root@bogon support-files]# chmod +x /etc/rc.d/init.d/mysqld
[root@bogon support-files]# chkconfig --add mysqld
启动mysql：
[root@tomcat mysql-8.0.28]# service mysqld start
Starting MySQL.. SUCCESS!
[root@bogon mysql-8.0.28]# mysql -uroot -p'l3XMt.f4q>Ya'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.28

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 

修改mysql的密码
mysql> alter user 'root'@'localhost' identified with mysql_native_password by '192412Wei';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

[root@bogon mysql-8.0.28]# service mysqld stop
Shutting down MySQL. SUCCESS! 
[root@tomcat mysql-8.0.28]# service mysqld start
Starting MySQL.. SUCCESS! 
[root@bogon mysql-8.0.28]# 

```





###### mysql8.0远程连接



使用Workbench等远程工具连接时报如下错误：

```
无法加载身份验证插件“ caching_sha2_password”

Authentication plugin 'caching_sha2_password' cannot be loaded:

dlopen(/usr/local/mysql/lib/plugin/caching_sha2_password.so, 2): image

not found
```



将MySQL 8.0配置为以mysql_native_password模式运行，则可能能够解决此问题。



创建zhangsan用户授予所有权限

```shell
mysql> create user 'zhangsan'@'%' identified with mysql_native_password by '192412Wei';
Query OK, 0 rows affected (0.01 sec)

mysql> grant all privileges on *.* to 'zhangsan'@'%';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

```

