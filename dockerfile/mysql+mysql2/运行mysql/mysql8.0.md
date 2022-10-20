### Mysql8.0的安装

官网下载mysql：https://dev.mysql.com/downloads/mysql/

[官网下载mysql](https://dev.mysql.com/downloads/mysql/)
[官网安装介绍](https://dev.mysql.com/doc/refman/8.0/en/binary-installation.html)

![image-20220420150700297](https://cdn.jsdelivr.net/gh/LostStata/pictures@main/Mysql/202204201507447.png)

#### 1、下载解压包

```
[root@bogon ~]# wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.28-linux-glibc2.12-x86_64.tar.xz
[root@bogon ~]# tar -xf mysql-8.0.28-linux-glibc2.12-i686.tar.xz -C /usr/local/
[root@bogon ~]# cd /usr/local/
[root@bogon local]# mv mysql-8.0.28-linux-glibc2.12-i686/ mysql-8.0.28
```

#### 2、清理环境

``` shell
[root@bogon local]# rpm -aq mariadb*
mariadb-libs-5.5.56-2.el7.x86_64
[root@bogon local]# yum remove -y `rpm -aq mariadb*`
```

#### 3、创建mysql用户

``` shell
[root@bogon local]# groupadd mysql
[root@bogon local]# useradd -r -g mysql mysql

```

#### 4、创建mysqldata目录

```shell
[root@bogon local]# mkdir /usr/local/mysql-8.0.28/data
[root@bogon local]# chown -R mysql:mysql /usr/local/mysql-8.0.28/
[root@bogon local]# chmod 750 /usr/local/mysql-8.0.28/
```

#### 5、配置my.cnf

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



#### 6、初始化mysql

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

#### 7、配置mysql环境变量

```shell
vi /etc/profile
添加如下
MYSQL_HOME="/usr/local/mysql-8.0.28"
PATH="$PATH:$MYSQL_HOME/bin"
[root@bogon mysql-8.0.28]# source /etc/profile

```

#### 8、配置mysql的server

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

#### 9、MySQL的其他配置

##### mysql的开机自启

``` shell
方法一：配置mysql的开启自启
[root@bogon support-files]# chmod +x /etc/rc.d/init.d/mysqld
[root@bogon support-files]# chkconfig --add mysqld
方法二：在/etc/rc.d/rc.local添加/etc/init.d/mysqld start
[root@bogon support-files]# cat /etc/rc.d/rc.local 
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this file.
#
# In contrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.

touch /var/lock/subsys/local

/etc/init.d/mysqld start
/sbin/ntpdate cn.pool.ntp.org
添加执行权限
[root@bogon support-files]# chmod +x /etc/rc.d/rc.local

```



##### mysql8.0远程连接

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

修改用户的权限为mysql_native_password

```shell
mysql>alter user 'root'@'localhost' identified with mysql_native_password by '192412Wei';
Query OK, 0 rows affected (0.02 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```





##### 登录权限设置：将root用户设置为任何ip可访问

``` shell
mysql> select user,host,authentication_string from user;
+---------------+-----------+-------------------------------------------+
| user          | host      | authentication_string                     |
+---------------+-----------+-------------------------------------------+
| root          | localhost | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
| mysql.session | localhost | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| mysql.sys     | localhost | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
+---------------+-----------+-------------------------------------------+
3 rows in set (0.00 sec)
mysql> update user set host = '%' where user = 'root';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
mysql> commit;
Query OK, 0 rows affected (0.00 sec)
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```



##### my.cnf参数详解

```shell
#客户端设置，即客户端默认的连接参数
[client]
#默认连接端口　　　　　　　　　　　　　　　　
port = 3306
#用于本地连接的socket套接字
socket = /data/mysqldata/3306/mysql.sock
#编码
default-character-set = utf8mb4

#服务端基本设置
[mysqld]
#MySQL监听端口
port = 3307

#为MySQL客户端程序和服务器之间的本地通讯指定一个套接字文件
socket = /data/mysqldata/3307/mysql.sock
#pid文件所在目录
pid-file = /data/mysqldata/3307/mysql.pid
#使用该目录作为根目录（安装目录）
basedir = /usr/local/mysql-5.7.11
#数据文件存放的目录
datadir = /data/mysqldata/3307/data
#MySQL存放临时文件的目录
tmpdir = /data/mysqldata/3307/tmp
#服务端默认编码（数据库级别）
character_set_server = utf8mb4
#服务端默认的比对规则，排序规则
collation_server = utf8mb4_bin
#MySQL启动用户
user = mysql

#This variable applies when binary logging is enabled. 
#It controls whether stored function creators can be trusted not to create stored functions that will cause 　
#unsafe events to be written to the binary log. 
#If set to 0 (the default), users are not permitted to create or alter stored functions unless they have the SUPER 
#privilege in addition to the CREATE ROUTINE or ALTER ROUTINE privilege. 开启了binlog后，必须设置这个值为1.主要是考虑binlog安全
log_bin_trust_function_creators = 1


#性能优化的引擎，默认关闭
performance_schema = 0

#secure_auth 为了防止低版本的MySQL客户端(<4.1)使用旧的密码认证方式访问高版本的服务器。MySQL 5.6.7开始secure_auth 默认为启用值1
secure_auth = 1

#开启全文索引
#ft_min_word_len = 1

#自动修复MySQL的myisam表
#myisam_recover

#明确时间戳默认null方式
explicit_defaults_for_timestamp


#计划任务（事件调度器）
event_scheduler
#跳过外部锁定;External-locking用于多进程条件下为MyISAM数据表进行锁定
skip-external-locking

#跳过客户端域名解析；当新的客户连接mysqld时，mysqld创建一个新的线程来处理请求。该线程先检查是否主机名在主机名缓存中。如果不在，线程试图解析主机名。
#使用这一选项以消除MySQL进行DNS解析的时间。但需要注意，如果开启该选项，则所有远程主机连接授权都要使用IP地址方式，否则MySQL将无法正常处理连接请求!
skip-name-resolve　　　　　　　　　　　　　　

#MySQL绑定IP
#bind-address = 127.0.0.1　　　　　　　　　　

#为了安全起见，复制环境的数据库还是设置--skip-slave-start参数，防止复制随着mysql启动而自动启动
skip-slave-start　　　　　　　　　　　　　　 

#The number of seconds to wait for more data from a master/slave connection before aborting the read. MySQL主从复制的时候，
slave_net_timeout = 30  　　　　　　　　　　　

#当Master和Slave之间的网络中断，但是Master和Slave无法察觉的情况下（比如防火墙或者路由问题）。
#Slave会等待slave_net_timeout设置的秒数后，才能认为网络出现故障，然后才会重连并且追赶这段时间主库的数据。
#1.用这三个参数来判断主从是否延迟是不准确的Slave_IO_Running,Slave_SQL_Running,Seconds_Behind_Master.还是用pt-heartbeat吧。
#2.slave_net_timeout不要用默认值，设置一个你能接受的延时时间。
#设定是否支持命令load data local infile。如果指定local关键词，则表明支持从客户主机读文件
local-infile = 0

#指定MySQL可能的连接数量。当MySQL主线程在很短的时间内得到非常多的连接请求，该参数就起作用，之后主线程花些时间（尽管很短）检查连接并且启动一个新线程。
#back_log参数的值指出在MySQL暂时停止响应新请求之前的短时间内多少个请求可以被存在堆栈中。
back_log = 1024

#sql_mode,定义了mysql应该支持的sql语法，数据校验等!  NO_AUTO_CREATE_USER：禁止GRANT创建密码为空的用户。
#sql_mode = 'PIPES_AS_CONCAT,ANSI_QUOTES,IGNORE_SPACE,NO_KEY_OPTIONS,NO_TABLE_OPTIONS,NO_FIELD_OPTIONS,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'

#NO_ENGINE_SUBSTITUTION 如果需要的存储引擎被禁用或未编译，可以防止自动替换存储引擎
sql_mode = NO_ENGINE_SUBSTITUTION,NO_AUTO_CREATE_USER　
#"NO_AUTO_CREATE_USER" 禁止GRANT创建密码为空的用户，在mysql8.0中已不适用
使用select @@sql_mode;在mysql中查询配置的sql_mode
mysql> select @@sql_mode;
+------------------------+
| @@sql_mode             |
+------------------------+
| NO_ENGINE_SUBSTITUTION |
+------------------------+
1 row in set (0.00 sec)

######常用sql_mode######
①ONLY_FULL_GROUP_BY
对于GROUP BY聚合操作，如果在SELECT中的列，没有在GROUP BY中出现，那么这个SQL是不合法的，因为列不在GROUP BY从句中
②NO_AUTO_VALUE_ON_ZERO
该值影响自增长列的插入。默认设置下，插入0或NULL代表生成下一个自增长值。如果用户希望插入的值为0，而该列又是自增长的，那么这个选项就有用了。
③STRICT_TRANS_TABLES
如果一个值不能插入到一个事务中，则中断当前的操作，对非事务表不做限制
④NO_ZERO_IN_DATE
不允许日期和月份为零
⑤NO_ZERO_DATE
mysql数据库不允许插入零日期，插入零日期会抛出错误而不是警告
⑥ERROR_FOR_DIVISION_BY_ZERO
在insert或update过程中，如果数据被零除，则产生错误而非警告。如果未给出该模式，那么数据被零除时Mysql返回NULL
⑦NO_AUTO_CREATE_USER
禁止GRANT创建密码为空的用户
⑧NO_ENGINE_SUBSTITUTION
如果需要的存储引擎被禁用或未编译，那么抛出错误。不设置此值时，用默认的存储引擎替代，并抛出一个异常
⑨PIPES_AS_CONCAT
将"||"视为字符串的连接操作符而非或运算符，这和Oracle数据库是一样是，也和字符串的拼接函数Concat想类似
⑩ANSI_QUOTES
不能用双引号来引用字符串，因为它被解释为识别符
##############################################

#索引块的缓冲区大小，对MyISAM表性能影响最大的一个参数.决定索引处理的速度，尤其是索引读的速度。默认值是16M，通过检查状态值Key_read_requests
#和Key_reads，可以知道key_buffer_size设置是否合理
key_buffer_size = 32M　　　　　　　　　　　　

#一个查询语句包的最大尺寸。消息缓冲区被初始化为net_buffer_length字节，但是可在需要时增加到max_allowed_packet个字节。
#该值太小则会在处理大包时产生错误。如果使用大的BLOB列，必须增加该值。
#这个值来限制server接受的数据包大小。有时候大的插入和更新会受max_allowed_packet 参数限制，导致写入或者更新失败。
max_allowed_packet = 512M

#线程缓存；主要用来存放每一个线程自身的标识信息，如线程id，线程运行时基本信息等等，我们可以通过 thread_stack 参数来设置为每一个线程栈分配多大的内存。
thread_stack = 256K　　　　　　　　　　　　　

#是MySQL执行排序使用的缓冲大小。如果想要增加ORDER BY的速度，首先看是否可以让MySQL使用索引而不是额外的排序阶段。
#如果不能，可以尝试增加sort_buffer_size变量的大小。
sort_buffer_size = 16M　　　　　　　　　　　 

#是MySQL读入缓冲区大小。对表进行顺序扫描的请求将分配一个读入缓冲区，MySQL会为它分配一段内存缓冲区。read_buffer_size变量控制这一缓冲区的大小。
#如果对表的顺序扫描请求非常频繁，并且你认为频繁扫描进行得太慢，可以通过增加该变量值以及内存缓冲区大小提高其性能。
read_buffer_size = 16M

#应用程序经常会出现一些两表（或多表）Join的操作需求，MySQL在完成某些 Join 需求的时候（all/index join），为了减少参与Join的“被驱动表”的 
#读取次数以提高性能，需要使用到 Join Buffer 来协助完成 Join操作。当 Join Buffer 太小，MySQL 不会将该 Buffer 存入磁盘文件， #而是先将Join Buffer中的结果集与需要 Join 的表进行 Join 操作， #然后清空 Join Buffer 中的数据，继续将剩余的结果集写入此 Buffer 中，如此往复。这势必会造成被驱动表需要被多次读取，成倍增加 IO 访问，降低效率。
join_buffer_size = 16M　　　　　　　　　　　

#是MySQL的随机读缓冲区大小。当按任意顺序读取行时(例如，按照排序顺序)，将分配一个随机读缓存区。进行排序查询时，MySQL会首先扫描一遍该缓冲，以避免磁盘搜索，
#提高查询速度，如果需要排序大量数据，可适当调高该值。但MySQL会为每个客户连接发放该缓冲空间，所以应尽量适当设置该值，以避免内存开销过大。
read_rnd_buffer_size = 32M　

#通信缓冲区在查询期间被重置到该大小。通常不要改变该参数值，但是如果内存不足，可以将它设置为查询期望的大小。
#（即，客户发出的SQL语句期望的长度。如果语句超过这个长度，缓冲区自动地被扩大，直到max_allowed_packet个字节。）　　　　　　　　　　　　　　　　　　　
net_buffer_length = 16K　　　　　　　　　　 

　　　　　　　　　　　　　　　　　　　　　  
#当对MyISAM表执行repair table或创建索引时，用以缓存排序索引；设置太小时可能会遇到” myisam_sort_buffer_size is too small”
myisam_sort_buffer_size = 128M　
#默认8M，当对MyISAM非空表执行insert … select/ insert … values(…),(…)或者load data infile时，使用树状cache缓存数据，每个thread分配一个；
#注：当对MyISAM表load 大文件时，调大　bulk_insert_buffer_size/myisam_sort_buffer_size/key_buffer_size会极大提升速度　　　　 
bulk_insert_buffer_size = 32M　　　　　　  
#thread_cahe_size线程池，线程缓存。用来缓存空闲的线程，以至于不被销毁，如果线程缓存在的空闲线程，需要重新建立新连接，
#则会优先调用线程池中的缓存，很快就能响应连接请求。每建立一个连接，都需要一个线程与之匹配。
thread_cache_size = 384　　　　　　　　   
#工作原理： 一个SELECT查询在DB中工作后，DB会把该语句缓存下来，当同样的一个SQL再次来到DB里调用时，DB在该表没发生变化的情况下把结果从缓存中返回给Client。
#在数据库写入量或是更新量也比较大的系统，该参数不适合分配过大。而且在高并发，写入量大的系统，建系把该功能禁掉。
query_cache_size = 0　　　　　　　　　    

#决定是否缓存查询结果。这个变量有三个取值：0,1,2，分别代表了off、on、demand。
query_cache_type = 0　　　　　　　　　    
　　　　　　　
#它规定了内部内存临时表的最大值，每个线程都要分配。（实际起限制作用的是tmp_table_size和max_heap_table_size的最小值。）
#如果内存临时表超出了限制，MySQL就会自动地把它转化为基于磁盘的MyISAM表，存储在指定的tmpdir目录下
tmp_table_size = 1024M
#独立的内存表所允许的最大容量.# 此选项为了防止意外创建一个超大的内存表导致永尽所有的内存资源.
max_heap_table_size = 512M  　　　　　　   
#mysql打开最大文件数
open_files_limit = 10240　　　　　　　　　 

#MySQL无论如何都会保留一个用于管理员（SUPER）登陆的连接，用于管理员连接数据库进行维护操作，即使当前连接数已经达到了max_connections。
#因此MySQL的实际最大可连接数为max_connections+1；
#这个参数实际起作用的最大值（实际最大可连接数）为16384，即该参数最大值不能超过16384，即使超过也以16384为准；
#增加max_connections参数的值，不会占用太多系统资源。系统资源（CPU、内存）的占用主要取决于查询的密度、效率等；
#该参数设置过小的最明显特征是出现”Too many connections”错误；
max_connections = 2000

#用来限制用户资源的，0不限制；对整个服务器的用户限制
max-user-connections = 0


#max_connect_errors是一个MySQL中与安全有关的计数器值，它负责阻止过多尝试失败的客户端以防止暴力破解密码的情况。max_connect_errors的值与性能并无太大关系。
#当此值设置为10时，意味着如果某一客户端尝试连接此MySQL服务器，但是失败（如密码错误等等）10次，则MySQL会无条件强制阻止此客户端连接。
max_connect_errors = 100000　　　　　　　　 


#表描述符缓存大小，可减少文件打开/关闭次数
table_open_cache = 5120　　　　　　　　　　

#interactive_time -- 指的是mysql在关闭一个交互的连接之前所要等待的秒数(交互连接如mysql gui tool中的连接）
interactive_timeout = 86400　　　　　　　　
#wait_timeout -- 指的是MySQL在关闭一个非交互的连接之前所要等待的秒数
wait_timeout = 86400　　　　　　　　　　　　

#二进制日志缓冲大小 
#我们知道InnoDB存储引擎是支持事务的，实现事务需要依赖于日志技术，为了性能，日志编码采用二进制格式。那么，我们如何记日志呢？有日志的时候，就直接写磁盘？
#可是磁盘的效率是很低的，如果你用过Nginx，，一般Nginx输出access log都是要缓冲输出的。因此，记录二进制日志的时候，我们是否也需要考虑Cache呢？
#答案是肯定的，但是Cache不是直接持久化，于是面临安全性的问题——因为系统宕机时，Cache中可能有残余的数据没来得及写入磁盘。因此，Cache要权衡，要恰到好处：
#既减少磁盘I/O，满足性能要求；又保证Cache无残留，及时持久化，满足安全要求。
binlog_cache_size = 16M

#开启慢查询
slow_query_log = 1

#超过的时间为1s；MySQL能够记录执行时间超过参数 long_query_time 设置值的SQL语句，默认是不记录的。
long_query_time = 1


#记录管理语句和没有使用index的查询记录
log-slow-admin-statements 
log-queries-not-using-indexes　　　　　　　


# *** Replication related settings ***
#在复制方面的改进就是引进了新的复制技术：基于行的复制。简言之，这种新技术就是关注表中发生变化的记录，而非以前的照抄 binlog 模式。
#从 MySQL 5.1.12 开始，可以用以下三种模式来实现：基于SQL语句的复制(statement-based replication, SBR)，基于行的复制(row-based replication, RBR)，混合模式复制(mixed-based replication, MBR)。相应地，binlog的格式也有三种：STATEMENT，ROW，MIXED。MBR 模式中，SBR 模式是默认的。
binlog_format = ROW

# 为每个session 最大可分配的内存，在事务过程中用来存储二进制日志的缓存。
#max_binlog_cache_size = 102400
#开启二进制日志功能，binlog数据位置
log-bin = /data/mysqldata/3307/binlog/mysql-bin
log-bin-index = /data/mysqldata/3307/binlog/mysql-bin.index
#relay-log日志记录的是从服务器I/O线程将主服务器的二进制日志读取过来记录到从服务器本地文件，
#然后SQL线程会读取relay-log日志的内容并应用到从服务器
#binlog传到备机被写道relaylog里，备机的slave sql线程从relaylog里读取然后应用到本地。
relay-log = /data/mysqldata/3307/relay/mysql-relay-bin
relay-log-index = /data/mysqldata/3307/relay/mysql-relay-bin.index  

#服务端ID，用来高可用时做区分
server_id = 100
#log_slave_updates是将从服务器从主服务器收到的更新记入到从服务器自己的二进制日志文件中。
log_slave_updates = 1　　　　　　　　　　 
#二进制日志自动删除的天数。默认值为0,表示“没有自动删除”。启动时和二进制日志循环时可能删除。
expire-logs-days = 15　　　　　　　　　　 
#如果二进制日志写入的内容超出给定值，日志就会发生滚动。你不能将该变量设置为大于1GB或小于4096字节。 默认值是1GB。
max_binlog_size = 512M　　　　　　　　　　    

#replicate-wild-ignore-table参数能同步所有跨数据库的更新，比如replicate-do-db或者replicate-ignore-db不会同步类似 
replicate-wild-ignore-table = mysql.%　　
#设定需要复制的Table
#replicate-wild-do-table = db_name.%
#复制时跳过一些错误;不要胡乱使用这些跳过错误的参数，除非你非常确定你在做什么。当你使用这些参数时候，MYSQL会忽略那些错误，
#这样会导致你的主从服务器数据不一致。
#slave-skip-errors = 1062,1053,1146


#这两个参数一般用在主主同步中，用来错开自增值, 防止键值冲突
auto_increment_offset = 1
auto_increment_increment = 2　　　　　　　　

#将中继日志的信息写入表:mysql.slave_realy_log_info
relay_log_info_repository = TABLE　　　　　
#将master的连接信息写入表：mysql.salve_master_info
master_info_repository = TABLE　　　　　　 
#中继日志自我修复；当slave从库宕机后，假如relay-log损坏了，导致一部分中继日志没有处理，则自动放弃所有未执行的relay-log，
#并且重新从master上获取日志，这样就保证了relay-log的完整性
relay_log_recovery = on　　　　　　　　　　
# *** innodb setting ***
#InnoDB 用来高速缓冲数据和索引内存缓冲大小。 更大的设置可以使访问数据时减少磁盘 I/O。
innodb_buffer_pool_size = 4G

#单独指定数据文件的路径与大小
innodb_data_file_path = ibdata1:1G:autoextend

#每次commit 日志缓存中的数据刷到磁盘中。通常设置为 1，意味着在事务提交前日志已被写入磁盘， 事务可以运行更长以及服务崩溃后的修复能力。
#如果你愿意减弱这个安全，或你运行的是比较小的事务处理，可以将它设置为 0 ，以减少写日志文件的磁盘 I/O。这个选项默认设置为 0。
innodb_flush_log_at_trx_commit = 0

#sync_binlog=n，当每进行n次事务提交之后，MySQL将进行一次fsync之类的磁盘同步指令来将binlog_cache中的数据强制写入磁盘。
#sync_binlog = 1000

#对于多核的CPU机器，可以修改innodb_read_io_threads和innodb_write_io_threads来增加IO线程，来充分利用多核的性能
innodb_read_io_threads = 8　　
innodb_write_io_threads = 8　　　　　　　　

#Innodb Plugin引擎开始引入多种格式的行存储机制，目前支持：Antelope、Barracuda两种。其中Barracuda兼容Antelope格式。
innodb_file_format = Barracuda

#限制Innodb能打开的表的数量
innodb_open_files = 65536
#开始碎片回收线程。这个应该能让碎片回收得更及时而且不影响其他线程的操作
innodb_purge_threads = 1　
#分布式事务
innodb_support_xa = FALSE　
#InnoDB 将日志写入日志磁盘文件前的缓冲大小。理想值为 1M 至 8M。大的日志缓冲允许事务运行时不需要将日志保存入磁盘而只到事务被提交(commit)。　
#因此，如果有大的事务处理，设置大的日志缓冲可以减少磁盘I/O。　　　　　
innodb_log_buffer_size = 256M

#日志组中的每个日志文件的大小(单位 MB)。如果 n 是日志组中日志文件的数目，那么理想的数值为 1M 至下面设置的缓冲池(buffer pool)大小的 1/n。较大的值，
#可以减少刷新缓冲池的次数，从而减少磁盘 I/O。但是大的日志文件意味着在崩溃时需要更长的时间来恢复数据。
innodb_log_file_size = 1G

#指定有三个日志组
innodb_log_files_in_group = 3

#在回滚(rooled back)之前，InnoDB 事务将等待超时的时间(单位 秒)
#innodb_lock_wait_timeout = 120

#innodb_max_dirty_pages_pct作用：控制Innodb的脏页在缓冲中在那个百分比之下，值在范围1-100,默认为90.这个参数的另一个用处：
#当Innodb的内存分配过大，致使swap占用严重时，可以适当的减小调整这个值，使达到swap空间释放出来。建义：这个值最大在90%，最小在15%。
#太大，缓存中每次更新需要致换数据页太多，太小，放的数据页太小，更新操作太慢。
innodb_max_dirty_pages_pct = 75　　　　　
#innodb_buffer_pool_size 一致 可以开启多个内存缓冲池，把需要缓冲的数据hash到不同的缓冲池中，这样可以并行的内存读写。
innodb_buffer_pool_instances = 4 　　　 

#这个参数据控制Innodb checkpoint时的IO能力
innodb_io_capacity = 500　　　　　　　　

#作用：使每个Innodb的表，有自已独立的表空间。如删除文件后可以回收那部分空间。
#分配原则：只有使用不使用。但ＤＢ还需要有一个公共的表空间。
innodb_file_per_table = 1

#当更新/插入的非聚集索引的数据所对应的页不在内存中时（对非聚集索引的更新操作通常会带来随机IO），会将其放到一个insert buffer中，
#当随后页面被读到内存中时，会将这些变化的记录merge到页中。当服务器比较空闲时，后台线程也会做merge操作
innodb_change_buffering = inserts　　　　

#该值影响每秒刷新脏页的操作，开启此配置后，刷新脏页会通过判断产生重做日志的速度来判断最合适的刷新脏页的数量；
innodb_adaptive_flushing = 1　　　　　　

#数据库事务隔离级别 ，读取提交内容
transaction-isolation = READ-COMMITTED　　

#innodb_flush_method这个参数控制着innodb数据文件及redo log的打开、刷写模式
#InnoDB使用O_DIRECT模式打开数据文件，用fsync()函数去更新日志和数据文件。
innodb_flush_method = O_DIRECT　　　　　　

#默认设置值为1.设置为0：表示Innodb使用自带的内存分配程序；设置为1：表示InnoDB使用操作系统的内存分配程序。
#innodb_use_sys_malloc = 1　　　　　　　　




[mysqldump]
#它强制 mysqldump 从服务器查询取得记录直接输出而不是取得所有记录后将它们缓存到内存中
quick　　　　　　　　　　　　　　　　　

#限制server接受的数据包大小;指代mysql服务器端和客户端在一次传送数据包的过程当中数据包的大小
max_allowed_packet = 512M   　　　　    　　
#TCP/IP和套接字通信缓冲区大小,创建长度达net_buffer_length的行
net_buffer_length = 16384   　　　　    　　


[mysql]
#auto-rehash是自动补全的意思
auto-rehash　　　　　　　　　　　　　　


#isamchk数据检测恢复工具
[isamchk]   　　　　　　　　　　　　　　
key_buffer = 256M
sort_buffer_size = 256M
read_buffer = 2M
write_buffer = 2M


#使用myisamchk实用程序来获得有关你的数据库桌表的信息、检查和修复他们或优化他们
[myisamchk]
key_buffer = 256M
sort_buffer_size = 256M
read_buffer = 2M
write_buffer = 2M


[mysqlhotcopy]
#mysqlhotcopy使用lock tables、flush tables和cp或scp来快速备份数据库.它是备份数据库或单个表最快的途径,完全属于物理备份,但只能用于备份MyISAM存储引擎和运行在数据库目录所在的机器上.
interactive-timeout

#与mysqldump备份不同,mysqldump属于逻辑备份,备份时是执行的sql语句.使用mysqlhotcopy命令前需要要安装相应
```

###### 核心参数含义

``` shell
innodb_buffer_pool 
# 注：缓冲池位于主内存中，InnoDB用它来缓存被访问过的表和索引文件，使常用数据可以直接在内存中被处理，从而提升处理速度；

innodb_buffer_pool_instance
# 注：MySQL5.6.6之后可以调整为多个。表示InnoDB缓冲区可以被划分为多个区域，也可以理解为把innodb_buffer_pool划分为多个实例，可以提高并发性，避免在高并发环境下，出现内存的争用问题；

innodb_data_file_path
# 注：该参数可以指定系统表空间文件的路径和ibdata1文件的大小。默认大小是10MB，这里建议调整为1GB

transaction_isolation
# 注：MySQL数据库的事务隔离级别有四种，分别为READ-UNCOMMITTED、READ-COMMITTED、REPEATABLE-READ和SERIALIZABLE。默认采用REPEATABLE-READ（可重复读）

innodb_log_buffer_size
# 注：是日志缓冲的大小，InnoDB改变数据的时候，它会把这次改动的记录先写到日志缓冲中

innodb_log_file_size
# 注：是指Redo log日志的大小，该值设置不宜过大也不宜过小，如果设置太大，实例恢复的时候需要较长时间，如果设置太小，会造成redo log 切换频繁，产生无用的I/O消耗，影响数据库性能

innodb_log_files_in_group
# 注：redo log文件组中日志文件的数量，默认情况下至少有2个

max_connections
# 该参数代表MySQL数据库的最大连接数

expire_logs_days
# 注：该参数代表binlog的过期时间，单位是天

slow_query_log
# 注：慢查询日志的开关，该参数等于1代表开启慢查询

long_query_time
# 注：慢查询的时间，某条SQL语句超过该参数设置的时间，就会记录到慢查询日志中。单位是秒

binlog_format
# 注：该参数代表二进制日志的格式。binlog格式有三种statement、row和mixed。生产环境中使用row这种格式更安全，不会出现跨库复制丢数据的情况

lower_case_table_names
# 注：表名是否区分大小的参数。默认是值为0。0代表区分大小写，1代表不区分大小写，以小写存储

interactive_timeout
# 注：是服务器关闭交互式连接前等待活动的时间,默认是28800s（8小时）

wait_timeout
# 注：是服务器关闭非交互式连接之前等待活动的时间，默认是28800s（8小时）

innodb_flush_method
# 注：这个参数影响InnoDB数据文件，redo log文件的打开刷写模式

log_queries_not_using_indexes
# 注：如果运行的SQL语句没有使用索引，则MySQL数据库同样会将这条SQL语句记录到慢查询日志文件中

```

除了核心参数之外，还有一些其他参数

###### 其他参数含义

``` shell
[client]
port = 3306
socket = /tmp/mysql.sock
[mysqld]
port = 3306
socket = /tmp/mysql.sock
basedir = /usr/local/mysql
datadir = /data/mysql
pid-file = /data/mysql/mysql.pid
user = mysql
bind-address = 0.0.0.0
server-id = 1 #表示是本机的序号为1,一般来讲就是master的意思

skip-name-resolve
# 禁止MySQL对外部连接进行DNS解析，使用这一选项可以消除MySQL进行DNS解析的时间。但需要注意，如果开启该选项，
# 则所有远程主机连接授权都要使用IP地址方式，否则MySQL将无法正常处理连接请求

#skip-networking
back_log = 600
# MySQL能有的连接数量。当主要MySQL线程在一个很短时间内得到非常多的连接请求，这就起作用，
# 然后主线程花些时间(尽管很短)检查连接并且启动一个新线程。back_log值指出在MySQL暂时停止回答新请求之前的短时间内多少个请求可以被存在堆栈中。
# 如果期望在一个短时间内有很多连接，你需要增加它。也就是说，如果MySQL的连接数据达到max_connections时，新来的请求将会被存在堆栈中，
# 以等待某一连接释放资源，该堆栈的数量即back_log，如果等待连接的数量超过back_log，将不被授予连接资源。
# 另外，这值（back_log）限于您的操作系统对到来的TCP/IP连接的侦听队列的大小。
# 你的操作系统在这个队列大小上有它自己的限制（可以检查你的OS文档找出这个变量的最大值），试图设定back_log高于你的操作系统的限制将是无效的。


max_connections = 1000
# MySQL的最大连接数，如果服务器的并发连接请求量比较大，建议调高此值，以增加并行连接数量，当然这建立在机器能支撑的情况下，因为如果连接数越多，介于MySQL会为每个连接提供连接缓冲区，就会开销越多的内存，所以要适当调整该值，不能盲目提高设值。可以过'conn%'通配符查看当前状态的连接数量，以定夺该值的大小。


max_connect_errors = 6000
# 对于同一主机，如果有超出该参数值个数的中断错误连接，则该主机将被禁止连接。如需对该主机进行解禁，执行：FLUSH HOST。


open_files_limit = 65535
# MySQL打开的文件描述符限制，默认最小1024;当open_files_limit没有被配置的时候，比较max_connections*5和ulimit -n的值，哪个大用哪个，
# 当open_file_limit被配置的时候，比较open_files_limit和max_connections*5的值，哪个大用哪个。

table_open_cache = 128
# MySQL每打开一个表，都会读入一些数据到table_open_cache缓存中，当MySQL在这个缓存中找不到相应信息时，才会去磁盘上读取。默认值64
# 假定系统有200个并发连接，则需将此参数设置为200*N(N为每个连接所需的文件描述符数目)；
# 当把table_open_cache设置为很大时，如果系统处理不了那么多文件描述符，那么就会出现客户端失效，连接不上

max_allowed_packet = 4M
# 接受的数据包大小；增加该变量的值十分安全，这是因为仅当需要时才会分配额外内存。例如，仅当你发出长查询或MySQLd必须返回大的结果行时MySQLd才会分配更多内存。
# 该变量之所以取较小默认值是一种预防措施，以捕获客户端和服务器之间的错误信息包，并确保不会因偶然使用大的信息包而导致内存溢出。

binlog_cache_size = 1M
# 一个事务，在没有提交的时候，产生的日志，记录到Cache中；等到事务提交需要提交的时候，则把日志持久化到磁盘。默认binlog_cache_size大小32K

max_heap_table_size = 8M
# 定义了用户可以创建的内存表(memory table)的大小。这个值用来计算内存表的最大行数值。这个变量支持动态改变

tmp_table_size = 16M
# MySQL的heap（堆积）表缓冲大小。所有联合在一个DML指令内完成，并且大多数联合甚至可以不用临时表即可以完成。
# 大多数临时表是基于内存的(HEAP)表。具有大的记录长度的临时表 (所有列的长度的和)或包含BLOB列的表存储在硬盘上。
# 如果某个内部heap（堆积）表大小超过tmp_table_size，MySQL可以根据需要自动将内存中的heap表改为基于硬盘的MyISAM表。还可以通过设置tmp_table_size选项来增加临时表的大小。也就是说，如果调高该值，MySQL同时将增加heap表的大小，可达到提高联接查询速度的效果

read_buffer_size = 2M
# MySQL读入缓冲区大小。对表进行顺序扫描的请求将分配一个读入缓冲区，MySQL会为它分配一段内存缓冲区。read_buffer_size变量控制这一缓冲区的大小。
# 如果对表的顺序扫描请求非常频繁，并且你认为频繁扫描进行得太慢，可以通过增加该变量值以及内存缓冲区大小提高其性能

read_rnd_buffer_size = 8M
# MySQL的随机读缓冲区大小。当按任意顺序读取行时(例如，按照排序顺序)，将分配一个随机读缓存区。进行排序查询时，
# MySQL会首先扫描一遍该缓冲，以避免磁盘搜索，提高查询速度，如果需要排序大量数据，可适当调高该值。但MySQL会为每个客户连接发放该缓冲空间，所以应尽量适当设置该值，以避免内存开销过大

sort_buffer_size = 8M
# MySQL执行排序使用的缓冲大小。如果想要增加ORDER BY的速度，首先看是否可以让MySQL使用索引而不是额外的排序阶段。
# 如果不能，可以尝试增加sort_buffer_size变量的大小

join_buffer_size = 8M
# 联合查询操作所能使用的缓冲区大小，和sort_buffer_size一样，该参数对应的分配内存也是每连接独享

thread_cache_size = 8
# 这个值（默认8）表示可以重新利用保存在缓存中线程的数量，当断开连接时如果缓存中还有空间，那么客户端的线程将被放到缓存中，
# 如果线程重新被请求，那么请求将从缓存中读取,如果缓存中是空的或者是新的请求，那么这个线程将被重新创建,如果有很多新的线程，
# 增加这个值可以改善系统性能.通过比较Connections和Threads_created状态的变量，可以看到这个变量的作用。(–>表示要调整的值)
# 根据物理内存设置规则如下：
# 1G  —> 8
# 2G  —> 16
# 3G  —> 32
# 大于3G  —> 64

query_cache_size = 8M
#MySQL的查询缓冲大小（从4.0.1开始，MySQL提供了查询缓冲机制）使用查询缓冲，MySQL将SELECT语句和查询结果存放在缓冲区中，
# 今后对于同样的SELECT语句（区分大小写），将直接从缓冲区中读取结果。根据MySQL用户手册，使用查询缓冲最多可以达到238%的效率。
# 通过检查状态值'Qcache_%'，可以知道query_cache_size设置是否合理：如果Qcache_lowmem_prunes的值非常大，则表明经常出现缓冲不够的情况，
# 如果Qcache_hits的值也非常大，则表明查询缓冲使用非常频繁，此时需要增加缓冲大小；如果Qcache_hits的值不大，则表明你的查询重复率很低，
# 这种情况下使用查询缓冲反而会影响效率，那么可以考虑不用查询缓冲。此外，在SELECT语句中加入SQL_NO_CACHE可以明确表示不使用查询缓冲

query_cache_limit = 2M
#指定单个查询能够使用的缓冲区大小，默认1M

key_buffer_size = 4M
#指定用于索引的缓冲区大小，增加它可得到更好处理的索引(对所有读和多重写)，到你能负担得起那样多。如果你使它太大，
# 系统将开始换页并且真的变慢了。对于内存在4GB左右的服务器该参数可设置为384M或512M。通过检查状态值Key_read_requests和Key_reads，
# 可以知道key_buffer_size设置是否合理。比例key_reads/key_read_requests应该尽可能的低，
# 至少是1:100，1:1000更好(上述状态值可以使用SHOW STATUS LIKE 'key_read%'获得)。注意：该参数值设置的过大反而会是服务器整体效率降低

ft_min_word_len = 4
# 分词词汇最小长度，默认4

transaction_isolation = REPEATABLE-READ
# MySQL支持4种事务隔离级别，他们分别是：
# READ-UNCOMMITTED, READ-COMMITTED, REPEATABLE-READ, SERIALIZABLE.
# 如没有指定，MySQL默认采用的是REPEATABLE-READ，ORACLE默认的是READ-
COMMITTED

log_bin = mysql-bin

binlog_format = mixed

expire_logs_days = 30 #超过30天的binlog删除

log_error = /data/mysql/mysql-error.log #错误日志路径

slow_query_log = 1

long_query_time = 1 #慢查询时间 超过1秒则为慢查询

slow_query_log_file = /data/mysql/mysql-slow.log

performance_schema = 0

explicit_defaults_for_timestamp

#lower_case_table_names = 1 #不区分大小写

skip-external-locking #MySQL选项以避免外部锁定。该选项默认开启

default-storage-engine = InnoDB #默认存储引擎

innodb_file_per_table = 1
# InnoDB为独立表空间模式，每个数据库的每个表都会生成一个数据空间
# 独立表空间优点：
# 1．每个表都有自已独立的表空间。
# 2．每个表的数据和索引都会存在自已的表空间中。
# 3．可以实现单表在不同的数据库中移动。
# 4．空间可以回收（除drop table操作处，表空不能自已回收）
# 缺点：
# 单表增加过大，如超过100G

# 结论：
# 共享表空间在Insert操作上少有优势。其它都没独立表空间表现好。当启用独立表空间时，请合理调整：innodb_open_files
innodb_open_files = 500
# 限制Innodb能打开的表的数据，如果库里的表特别多的情况，请增加这个。这个值默认是300

innodb_buffer_pool_size = 64M
# InnoDB使用一个缓冲池来保存索引和原始数据, 不像MyISAM.
# 这里你设置越大,你在存取表里面数据时所需要的磁盘I/O越少.
# 在一个独立使用的数据库服务器上,你可以设置这个变量到服务器物理内存大小的80%
# 不要设置过大,否则,由于物理内存的竞争可能导致操作系统的换页颠簸.
# 注意在32位系统上你每个进程可能被限制在 2-3.5G 用户层面内存限制,
# 所以不要设置的太高.

innodb_write_io_threads = 4
innodb_read_io_threads = 4
# innodb使用后台线程处理数据页上的读写 I/O(输入输出)请求,根据你的 CPU 核数来更改,默认是4
# 注:这两个参数不支持动态改变,需要把该参数加入到my.cnf里，修改完后重启MySQL服务,允许值的范围从 1-64

innodb_thread_concurrency = 0
# 默认设置为 0,表示不限制并发数，这里推荐设置为0，更好去发挥CPU多核处理能力，提高并发量

innodb_purge_threads = 1
# InnoDB中的清除操作是一类定期回收无用数据的操作。在之前的几个版本中，清除操作是主线程的一部分，这意味着运行时它可能会堵塞其它的数据库操作。
# 从MySQL5.5.X版本开始，该操作运行于独立的线程中,并支持更多的并发数。用户可通过设置innodb_purge_threads配置参数来选择清除操作是否使用单
# 独线程,默认情况下参数设置为0(不使用单独线程),设置为 1 时表示使用单独的清除线程。建议为1

innodb_flush_log_at_trx_commit = 2
# 0：如果innodb_flush_log_at_trx_commit的值为0,log buffer每秒就会被刷写日志文件到磁盘，提交事务的时候不做任何操作（执行是由mysql的master thread线程来执行的。
# 主线程中每秒会将重做日志缓冲写入磁盘的重做日志文件(REDO LOG)中。不论事务是否已经提交）默认的日志文件是ib_logfile0,ib_logfile1
# 1：当设为默认值1的时候，每次提交事务的时候，都会将log buffer刷写到日志。
# 2：如果设为2,每次提交事务都会写日志，但并不会执行刷的操作。每秒定时会刷到日志文件。要注意的是，并不能保证100%每秒一定都会刷到磁盘，这要取决于进程的调度。
# 每次事务提交的时候将数据写入事务日志，而这里的写入仅是调用了文件系统的写入操作，而文件系统是有 缓存的，所以这个写入并不能保证数据已经写入到物理磁盘
# 默认值1是为了保证完整的ACID。当然，你可以将这个配置项设为1以外的值来换取更高的性能，但是在系统崩溃的时候，你将会丢失1秒的数据。
# 设为0的话，mysqld进程崩溃的时候，就会丢失最后1秒的事务。设为2,只有在操作系统崩溃或者断电的时候才会丢失最后1秒的数据。InnoDB在做恢复的时候会忽略这个值。

# 总结
# 设为1当然是最安全的，但性能页是最差的（相对其他两个参数而言，但不是不能接受）。如果对数据一致性和完整性要求不高，完全可以设为2，如果只最求性能，例如高并发写的日志服务器，设为0来获得更高性能

innodb_log_buffer_size = 2M
# 此参数确定些日志文件所用的内存大小，以M为单位。缓冲区更大能提高性能，但意外的故障将会丢失数据。MySQL开发人员建议设置为1－8M之间

innodb_log_file_size = 32M
# 此参数确定数据日志文件的大小，更大的设置可以提高性能，但也会增加恢复故障数据库所需的时间

innodb_log_files_in_group = 3
# 为提高性能，MySQL可以以循环方式将日志文件写到多个文件。推荐设置为3

innodb_max_dirty_pages_pct = 90
# innodb主线程刷新缓存池中的数据，使脏数据比例小于90%

innodb_lock_wait_timeout = 120 
# InnoDB事务在被回滚之前可以等待一个锁定的超时秒数。InnoDB在它自己的锁定表中自动检测事务死锁并且回滚事务。InnoDB用LOCK TABLES语句注意到锁定设置。默认值是50秒

bulk_insert_buffer_size = 8M
# 批量插入缓存大小， 这个参数是针对MyISAM存储引擎来说的。适用于在一次性插入100-1000+条记录时， 提高效率。默认值是8M。可以针对数据量的大小，翻倍增加。

myisam_sort_buffer_size = 8M
# MyISAM设置恢复表之时使用的缓冲区的尺寸，当在REPAIR TABLE或用CREATE INDEX创建索引或ALTER TABLE过程中排序 MyISAM索引分配的缓冲区

myisam_max_sort_file_size = 10G
# 如果临时文件会变得超过索引，不要使用快速排序索引方法来创建一个索引。注释：这个参数以字节的形式给出

myisam_repair_threads = 1
# 如果该值大于1，在Repair by sorting过程中并行创建MyISAM表索引(每个索引在自己的线程内) 

interactive_timeout = 28800
# 服务器关闭交互式连接前等待活动的秒数。交互式客户端定义为在mysql_real_connect()中使用CLIENT_INTERACTIVE选项的客户端。默认值：28800秒（8小时）

wait_timeout = 28800
# 服务器关闭非交互连接之前等待活动的秒数。在线程启动时，根据全局wait_timeout值或全局interactive_timeout值初始化会话wait_timeout值，
# 取决于客户端类型(由mysql_real_connect()的连接选项CLIENT_INTERACTIVE定义)。参数默认值：28800秒（8小时）
# MySQL服务器所支持的最大连接数是有上限的，因为每个连接的建立都会消耗内存，因此我们希望客户端在连接到MySQL Server处理完相应的操作后，
# 应该断开连接并释放占用的内存。如果你的MySQL Server有大量的闲置连接，他们不仅会白白消耗内存，而且如果连接一直在累加而不断开，
# 最终肯定会达到MySQL Server的连接上限数，这会报'too many connections'的错误。对于wait_timeout的值设定，应该根据系统的运行情况来判断。
# 在系统运行一段时间后，可以通过show processlist命令查看当前系统的连接状态，如果发现有大量的sleep状态的连接进程，则说明该参数设置的过大，
# 可以进行适当的调整小些。要同时设置interactive_timeout和wait_timeout才会生效。

[mysqldump]
quick
max_allowed_packet = 16M #服务器发送和接受的最大包长度
[myisamchk]
key_buffer_size = 8M
sort_buffer_size = 8M
read_buffer = 4M
write_buffer = 4M

```

##### 写一个简单的模板

``` shell
# 简单模板如下：
[client]
port = 3306
socket = /tmp/mysql.sock
default-character-set = utf8mb4
[mysqld]
user = mysql
basedir=/usr/local/mysql-8.0.28
datadir=/usr/local/mysql-8.0.28/data
port = 3306 
socket=/tmp/mysql.sock
pid-file=/usr/local/mysql-8.0.28/data/mysql.pid
tmpdir = /tmp 
skip_name_resolve = 1
symbolic-links=0
max_connections = 2000
group_concat_max_len = 1024000
#sql_mode =NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
sql-mode="NO_ENGINE_SUBSTITUTION"
lower_case_table_names = 1
log_timestamps=SYSTEM
character-set-server = utf8
interactive_timeout = 1800 
wait_timeout = 1800
max_allowed_packet = 32M
binlog_cache_size = 4M
sort_buffer_size = 2M
read_buffer_size = 4M
join_buffer_size = 4M
tmp_table_size = 256M
max_heap_table_size = 256M
max_length_for_sort_data = 8096
#logs
server-id = 1003306
log-error=/usr/local/mysql-8.0.28/data/mysql.log
slow_query_log = 1
slow_query_log_file = /usr/local/mysql-8.0.28/data/slow.log
long_query_time = 3
log-bin = /usr/local/mysql-8.0.28/data/binlog
binlog_format = row
expire_logs_days = 15
log_bin_trust_function_creators = 1
relay-log = /usr/local/mysql-8.0.28/data/relay-bin
relay-log-recovery = 1 
relay_log_purge = 1 
#innodb 
innodb_file_per_table = 1
innodb_log_buffer_size = 16M
innodb_log_file_size = 256M
innodb_log_files_in_group = 2
innodb_io_capacity = 2000
innodb_io_capacity_max = 4000
innodb_flush_neighbors = 0
innodb_flush_method = O_DIRECT
innodb_autoinc_lock_mode = 2
innodb_read_io_threads = 8
innodb_write_io_threads = 8
innodb_buffer_pool_size = 2G




```



##### 后续问题补充

``` 
################### 问题一：执行mysql导入时，由于表数据过多报错；
[root@bogon support-files]# mysql -uroot -proot < full_20220420.sql 
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 1114 (HY000) at line 10692: The table 't_user' is full
[root@bogon support-files]#

################### 解决方法，打开mysql配置文件，修改如下2个配置，然后执行service mysql restart 重启mysql服务，再重新导入
vim /etc/my.cnf
tmp_table_size = 256M
max_heap_table_size = 256M

```



##### mysql的备份与恢复

``` shell
1.导出单表
[root@bogon ~]# mysqldump -uroot -p datax_web job_log_report > datax_web.sql
[root@bogon ~]# delete from job_log_report;
2.导出表结构
[root@bogon ~]# mysqldump -uroot -p -d datax_web job_log_report  > job_log_report.sql
3.导出不包含删除语句脚本
[root@bogon ~]# mysqldump -uroot -p -d [--skip-add-drop-table] datax_web job_log_report > job_log_report.sql
恢复
[root@bogon ~]# mysql -uroot -p datax_web < jog_log_report.sql 

#备份整个库
mysqldump -uroot -p 'your password' datax_web > datax_web.sql
进入数据库，创建需要的库；
#恢复库
1、mysql
mysql -uroot -p 'your password' datax_web < /root/datax_web.sql
2、source
#创建被恢复的数据库（root用户）
mysql> create database datax_web;
mysql> use datax_web;
mysql> source /root/datax_web.sql;  # datax_web.sql 可以指定完整路径
```



##### mysqldump解释

```shell
MySQLdump是MySQL自带的导出数据工具，通常我们用它来导出MySQL数据，但是有时候我们需要导出MySQL数据库中某个表的部分数据，这时该怎么办呢？

mysqldump命令中带有一个 --where/-w 参数，它用来设定数据导出的条件，使用方式和SQL查询命令中中的where基本上相同，有了它，我们就可以从数据库中导出你需要的那部分数据了。

命令格式如下：

mysqldump -u用户名 -p密码 数据库名 表名 --where="筛选条件" > 导出文件路径

例子：

从meteo数据库的sdata表中导出sensorid=11 且 fieldid=0的数据到 /home/xyx/Temp.sql 这个文件中

mysqldump -uroot -p123456 meteo sdata --where=" sensorid=11 and fieldid=0" > /home/xyx/Temp.sql

另外你还可以直接导出 文本文件*.txt

mysqldump -uroot -p123456 meteo sdata --where=" sensorid=11 and fieldid=0" > /home/xyx/Temp.txt

///

以下是 mysqldump 的一些使用参数

备份数据库

#mysqldump 数据库名 >数据库备份名

#mysqldump -A -u用户名 -p密码 数据库名>数据库备份名

#mysqldump -d -A --add-drop-table -uroot -p >xxx.sql

1.导出结构不导出数据

mysqldump -d 数据库名 -uroot -p > xxx.sql

2.导出数据不导出结构

mysqldump -t 数据库名 -uroot -p > xxx.sql

3.导出数据和表结构

mysqldump 数据库名 -uroot -p > xxx.sql

4.导出特定表的结构

mysqldump -uroot -p -B数据库名 --table 表名 > xxx.sql

#mysqldump [OPTIONS] database [tables]

mysqldump支持下列选项：

--add-locks

在每个表导出之前增加LOCK TABLES并且之后UNLOCK TABLE。(为了使得更快地插入到MySQL)。

--add-drop-table

在每个create语句之前增加一个drop table。

--allow-keywords

允许创建是关键词的列名字。这由表名前缀于每个列名做到。

-c, --complete-insert

使用完整的insert语句(用列名字)。

-C, --compress

如果客户和服务器均支持压缩，压缩两者间所有的信息。

--delayed

用INSERT DELAYED命令插入行。

-e, --extended-insert

使用全新多行INSERT语法。(给出更紧缩并且更快的插入语句)

-#, --debug[=option_string]

跟踪程序的使用(为了调试)。

--help

显示一条帮助消息并且退出。

--fields-terminated-by=...

--fields-enclosed-by=...

--fields-optionally-enclosed-by=...

--fields-escaped-by=...

--fields-terminated-by=...

这些选择与-T选择一起使用，并且有相应的LOAD DATA INFILE子句相同的含义。

LOAD DATA INFILE语法。

-F, --flush-logs

在开始导出前，洗掉在MySQL服务器中的日志文件。

-f, --force,

即使我们在一个表导出期间得到一个SQL错误，继续。

-h, --host=..

从命名的主机上的MySQL服务器导出数据。缺省主机是localhost。

-l, --lock-tables.

为开始导出锁定所有表。

-t, --no-create-info

不写入表创建信息(CREATE TABLE语句)

-d, --no-data

不写入表的任何行信息。如果你只想得到一个表的结构的导出，这是很有用的！

--opt

同--quick --add-drop-table --add-locks --extended-insert --lock-tables。

应该给你为读入一个MySQL服务器的尽可能最快的导出。

-pyour_pass, --password[=your_pass]

与服务器连接时使用的口令。如果你不指定“=your_pass”部分，mysqldump需要来自终端的口令。

-P port_num, --port=port_num

与一台主机连接时使用的TCP/IP端口号。(这用于连接到localhost以外的主机，因为它使用 Unix套接字。)

-q, --quick

不缓冲查询，直接导出至stdout；使用mysql_use_result()做它。

-S /path/to/socket, --socket=/path/to/socket

与localhost连接时(它是缺省主机)使用的套接字文件。

-T, --tab=path-to-some-directory

对于每个给定的表，创建一个table_name.sql文件，它包含SQL CREATE 命令，和一个table_name.txt文件，它包含数据。注意：这只有在mysqldump运行在mysqld守护进程运行的同一台机器上的时候才工作。.txt文件的格式根据--fields-xxx和 --lines--xxx选项来定。

-u user_name, --user=user_name

与服务器连接时，MySQL使用的用户名。缺省值是你的Unix登录名。

-O var=option, --set-variable var=option设置一个变量的值。可能的变量被列在下面。

-v, --verbose

冗长模式。打印出程序所做的更多的信息。

-V, --version

打印版本信息并且退出。

-w, --where='where-condition'

只导出被选择了的记录；注意引号是强制的！

"--where=user='jimf'" "-wuserid>1" "-wuserid<1"

导入数据：

由于mysqldump导出的是完整的SQL语句，所以用mysql客户程序很容易就能把数据导入了：

#mysql 数据库名 < 文件名

or:

#show databases;

然后选择被导入的数据库：

#use 'your database'；

#source /tmp/xxx.sql;
```



##### xtrabackup工具

1、下载

```shell
wget https://downloads.percona.com/downloads/percona-release/percona-release-1.0-9/redhat/percona-release-1.0-9.noarch.rpm
rpm -ivh percona-release-1.0-9.noarch.rpm
yum clean all && yum makecache
```

2、安装

``` shell
[root@bongo ~]# yum list | grep percona-xtrabackup*
percona-xtrabackup-24.x86_64             2.4.24-1.el7                   @percona-release-x86_64
percona-xtrabackup.x86_64                2.3.10-1.el7                   percona-release-x86_64
percona-xtrabackup-22.x86_64             2.2.13-1.el7                   percona-release-x86_64
percona-xtrabackup-22-debuginfo.x86_64   2.2.13-1.el7                   percona-release-x86_64
percona-xtrabackup-24-debuginfo.x86_64   2.4.24-1.el7                   percona-release-x86_64
percona-xtrabackup-80.x86_64             8.0.27-19.1.el7                percona-release-x86_64
percona-xtrabackup-80-debuginfo.x86_64   8.0.27-19.1.el7                percona-release-x86_64
percona-xtrabackup-debuginfo.x86_64      2.3.10-1.el7                   percona-release-x86_64
percona-xtrabackup-test.x86_64           2.3.10-1.el7                   percona-release-x86_64
percona-xtrabackup-test-22.x86_64        2.2.13-1.el7                   percona-release-x86_64
percona-xtrabackup-test-24.x86_64        2.4.24-1.el7                   percona-release-x86_64
percona-xtrabackup-test-80.x86_64        8.0.27-19.1.el7                percona-release-x86_64
[root@bongo ~]# yum -y install percona-xtrabackup-24.x86_64
```



创建备份用户

``` shell
mysql>CREATE USER 'bkpuser'@'localhost' IDENTIFIED with mysql_native_password BY '123456';
mysql>REVOKE ALL PRIVILEGES ON *.* FROM 'bkpuser'@'localhost';
mysql>REVOKE GRANT OPTION ON *.* FROM 'bkpuser'@'localhost';
mysql>GRANT RELOAD,LOCK TABLES,REPLICATION CLIENT ON *.* TO 'bkpuser'@'localhost';
mysql>FLUSH PRIVILEGES;
```



创建用户配置权限

```shell
mysql> create user 'backup'@'localhost' identified with mysql_native_password by "123456";
Query OK, 0 rows affected (0.04 sec)

mysql> REVOKE ALL PRIVILEGES on *.*  FROM "backup"@'localhost';
Query OK, 0 rows affected (0.01 sec)

mysql> REVOKE GRANT OPTION  on *.*  FROM "backup"@'localhost';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> GRANT RELOAD,LOCK TABLES,REPLICATION CLIENT ON *.* TO 'backup'@'localhost';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
```



#备份命令

innobackupex  --defaults-file=/etc/my.cnf --user=backup --password=123456 /data/mysqldata