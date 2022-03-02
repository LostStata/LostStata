# 源码安装mysql5.7

## 1、清理环境：

> 如是新环境可不执行删除用户及文件配置

#yum erase mariadb mariadb-server mariadb-libs mariadb-devel -y

#userdel  -r   mysql

#rm  -rf  /etc/my*

#rm  -rf  /var/lib/mysql

## 2、创建mysql用户

[root@mysql-server ~]# useradd -r mysql -M -s /bin/false

## 3、从官网下载tar包

#yum  -y  install  wget

#wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-boost-5.7.27.tar.gz

## 4、安装编译工具

#yum -y install ncurses ncurses-devel openssl-devel bison gcc gcc-c++ make

#yum -y install  cmake

## 5、创建mysql目录

[root@mysql-server ~]# mkdir -p /usr/local/{data,mysql,log}

## 6、解压

[root@mysql-server ~]# tar xzvf mysql-boost-5.7.27.tar.gz -C /usr/local/

> 注:如果安装的MySQL5.7及以上的版本，在编译安装之前需要安装boost,因为高版本mysql需要boots库的安装才可以正常运行。否则会报CMake Error at cmake/boost.cmake:81错误,安装包里面自带boost包

## 7、编译安装

> cd 解压的mysql目录

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

提示：boost也可以使用如下指令自动下载，如果不下载boost压缩包，把下面的这一条添加到配置中第二行
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

[root@mysql-server mysql-5.7.27]# make && make install
如果安装出错，想重新安装：
 不用重新解压，只需要删除安装目录中的缓存文件CMakeCache.txt

## 8、初始化

[root@mysql-server mysql-5.7.27]# cd /usr/local/mysql
[root@mysql-server mysql]# chown -R mysql.mysql .
[root@mysql-server mysql]# ./bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data   

---初始化完成之后，一定要记住提示最后的密码用于登陆或者修改密码

qtwcNAX5g#X(

![1595835348488](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1595835348488.png)
初始化,只需要初始化一次

[root@mysql-server ~]# vim /etc/my.cnf  ---添加如下内容
[mysqld]
basedir=/usr/local/mysql   #指定安装目录
datadir=/usr/local/mysql/data  #指定数据存放目录

## 9、启动mysql

[root@mysql-server ~]# cd /usr/local/mysql
[root@mysql-server mysql]# ./bin/mysqld_safe --user=mysql &

![1595835444499](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1595835444499.png)

## 10、登录mysql

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

## 11、修改密码

[root@mysql-server mysql]# /usr/local/mysql/bin/mysqladmin -u root -p'GP9TKGgY9i/8' password '192412Wei'
mysqladmin: [Warning] Using a password on the command line interface can be
insecure.
Warning: Since password will be sent to server in plain text, use ssl connection
to ensure password safety.

## 12、添加环境变量

[root@mysql-server mysql]# vim /etc/profile  ---添加如下
PATH=$PATH:$HOME/bin:/usr/local/mysql/bin
[root@mysql-server mysql]# source /etc/profile
之后就可以在任何地方使用mysql命令登陆Mysql服务器：

## 13、配置mysqld服务的管理工具：

[root@mysql-server mysql]# cd /usr/local/mysql/support-files/
[root@mysql-server support-files]# cp mysql.server /etc/init.d/mysqld
[root@mysql-server support-files]# chkconfig --add mysqld
[root@mysql-server support-files]# chkconfig mysqld on
先将原来的进程杀掉
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

数据库编译安装完成

## 14、mysql脚本简写

> shell中使用

``` shell
#!/bin/bash
#开头写set -e
set -e
#结尾写set +e 判断执行失败退出
set +e
```

> mysql脚本如下：

``` shell
#!/usr/bin/bash

echo '清理mariadb环境'
yum erase mariadb mariadb-server mariadb-libs mariadb-devel -y  &> /dev/null

echo '创建新的mysql用户'
userdel -r mysql
useradd -r mysql -M -s /bin/false

echo '下载mysql'
yum  -y  install  wget &> /dev/null
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-boost-5.7.27.tar.gz &> /dev/null
if [ $? -eq 0 ]; then
     echo "mysql-boot下载成功"  >> /root/init.log
else
     echo "mysql-boot下载失败"  >> /root/init.log
fi


echo '安装mysql所需的环境'
yum -y install ncurses ncurses-devel openssl-devel bison gcc gcc-c++ make cmake &> /dev/null

echo '创建所需的目录'
mkdir -p /usr/local/{data,mysql,log}

echo '解压mysql程序'
tar xzvf mysql-boost-5.7.27.tar.gz -C /usr/local/ &> /dev/null

echo '预编译'
cd /usr/local/mysql-5.7.27/
cmake . \
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
if [ $? -eq 0 ]; then
     echo "mysql预编译完成"  >> /root/init.log
else
     echo "mysql预编译失败"  >> /root/init.log
fi

echo '编译与安装'
make && make install
if [ $? -eq 0 ]; then
     echo "编译与安装完成"  >> /root/init.log
else
     echo "编译与安装失败"  >> /root/init.log
fi

echo '初始化mysql，保存文件在root下的init.log中'
cd /usr/local/mysql
chown -R mysql.mysql .
 ./bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data  >> /root/init.log
if [ $? -eq 0 ]; then
     echo "初始化成功"  >> /root/init.log
else
     echo "初始化失败"  >> /root/init.log
fi

echo "配置mysql配置文件"
cat >/etc/my.cnf<<EOF
[mysqld]
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
EOF

echo "配置环境变量"
echo  'PATH=$PATH:$HOME/bin:/usr/local/mysql/bin' >> /etc/profile
source /etc/profile

echo "配置mysql管理工具,并开机自启"
cd /usr/local/mysql/support-files/
cp mysql.server /etc/init.d/mysqld
chkconfig --add mysqld
chkconfig mysqld on

echo "启动mysql"
/etc/init.d/mysqld start

echo "查看端口"
yum  -y  install  net-tools
netstat  -lntp  |  grep  mysql   >>  /root/init.log

echo "关闭mysql"
/etc/init.d/mysqld stop
```















