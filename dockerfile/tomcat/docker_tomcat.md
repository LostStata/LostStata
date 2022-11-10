##### **1、下载所需的源码包**

```
wget https://dlcdn.apache.org/tomcat/tomcat-8/v8.5.73/bin/apache-tomcat-8.5.73.tar.gz

wget  https://repo.huaweicloud.com/java/jdk/8u181-b13/jdk-8u181-linux-x64.tar.gz
```

##### **2、编写dockerfile后创建所需的镜像**

``` shell
docker build -t tomcat:8.5.1 .
```

##### **3、创建需要挂载的目录**

```
mkdir -p /data/tomcat/{tomcat01,tomcat02,tomcat03}/{conf,webapps}
```

##### **4、将源文件复制到tomcat01、tomcat02、tomcat03下的挂载目录**

```
tar -xf apache-tomcat-8.5.73.tar.gz -C /data/tomcat/
cp  -r  apache-tomcat-8.5.73/conf/server.xml  /data/tomcat/tomcat01/conf/
cp  -r  apache-tomcat-8.5.73/conf/server.xml  /data/tomcat/tomcat02/conf/
cp  -r  apache-tomcat-8.5.73/conf/server.xml  /data/tomcat/tomcat03/conf/
cp  -r  apache-tomcat-8.5.73/webapps/ROOT/*  /data/tomcat/tomcat01/webapps/ROOT/
cp  -r  apache-tomcat-8.5.73/webapps/ROOT/*  /data/tomcat/tomcat02/webapps/ROOT/
cp  -r  apache-tomcat-8.5.73/webapps/ROOT/*  /data/tomcat/tomcat03/webapps/ROOT/
```

##### **5、docker run**

###### 1、开放外部指定端口  *参数：-p  开放端口：容器端口*

```
docker run -d -p 8091:8080  -p 8412:8443 --name tomcat_empone --restart=always    -v /data/tomcat/tomcat01/conf/server.xml:/usr/local/tomcat/conf/server.xml   -v /data/tomcat/tomcat01/webapps:/usr/local/tomcat/webapps  tomcat:8.5.1

docker run -d -p 8092:8080  -p 8413:8443 --name tomcat_emptwo --restart=always     -v /data/tomcat/tomcat02/conf/server.xml:/usr/local/tomcat/conf/server.xml     -v /data/tomcat/tomcat02/webapps:/usr/local/tomcat/webapps  tomcat:8.5.1

docker run -d -p 8093:8080  -p  8414:8443 --name tomcat_empthree --restart=always   -v /data/tomcat/tomcat03/conf/server.xml:/usr/local/tomcat/conf/server.xml    -v /data/tomcat/tomcat03/webapps:/usr/local/tomcat/webapps  tomcat:8.5.1
```

###### 2、开放随机端口  *参数：-P*

```
docker run -d  -P  --name tomcat_empone --restart=always    -v /data/tomcat/tomcat01/conf/server.xml:/usr/local/tomcat/conf/server.xml   -v /data/tomcat/tomcat01/webapps:/usr/local/tomcat/webapps  tomcat:8.5.1

docker run -d  -P  --name tomcat_emptwo --restart=always     -v /data/tomcat/tomcat02/conf/server.xml:/usr/local/tomcat/conf/server.xml     -v /data/tomcat/tomcat02/webapps:/usr/local/tomcat/webapps  tomcat:8.5.1

docker run -d  -P  --name tomcat_empthree --restart=always   -v   /data/tomcat/tomcat03/conf/server.xml:/usr/local/tomcat/conf/server.xml    -v /data/tomcat/tomcat03/webapps:/usr/local/tomcat/webapps  tomcat:8.5.1
```

###### 3、创建指定ip的容器  *参数：docker  network  create*

```
创建一个新的bridge网络，默认为bridge模式
docker  network  create  --driver  bridge  --subnet  172.16.0.0/16  --gateway  172.16.0.1  emp

解析：
--driver bridge 表示使用桥接模式
--subnet 172.16.0.0/16 表示子网ip 可以分配 172.16.1.2 到 172.16.255.255
--gateway 172.16.0.1 表示网关
emp 表示网络名

docker run -d  -P  --net  emp  --ip  172.16.125.12  --name tomcat_empone --restart=always    -v /data/tomcat/tomcat01/conf/server.xml:/usr/local/tomcat/conf/server.xml   -v /data/tomcat/tomcat01/webapps:/usr/local/tomcat/webapps  tomcat:8.5.1

docker run -d  -P  --net  emp  --ip  172.16.125.13  --name tomcat_emptwo --restart=always     -v /data/tomcat/tomcat02/conf/server.xml:/usr/local/tomcat/conf/server.xml     -v /data/tomcat/tomcat02/webapps:/usr/local/tomcat/webapps  tomcat:8.5.1

docker run -d  -P  --net  emp  --ip  172.16.125.14  --name tomcat_empthree --restart=always   -v /data/tomcat/tomcat03/conf/server.xml:/usr/local/tomcat/conf/server.xml    -v /data/tomcat/tomcat03/webapps:/usr/local/tomcat/webapps  tomcat:8.5.1
```

**注释：**

tomcat修改挂载的文件，容器内文件不能修改成功，原因是：挂载进docker的文件，实际上是docker记住了一个iNode，他可以通过这个iNode找到block，也就是实际的文件信息.如果是用 > 追加重定向写入文件，是可以同步到docker的,但是如果是rm 重命名的，文件的iNode就改变了，但是docker中的iNode还是指向了之前的磁盘位置，所以文件没有改变

同样的还有vim ，vim 文件的时候是基于现有的文件copy了一份，同级目录下会有一个，开头swp结尾的文件，当你保存退出的时候，vim 会删掉源文件，将这个文件重命名为源文件的名字，iNode自然也就改变了。
建议固定容器ip，ip发生改变后，需要修改容器内部nginx的配置ip地址

**备注：**

如需更改其他文件，在容器挂载的时候挂载出来，进行配置

##### 镜像文件

编写文件名为Dockerfile

```
FROM centos:7.2.1511

ADD jdk-8u181-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-8.5.73.tar.gz /usr/local/

ENV MYPATH /usr/local/
WORKDIR $MYPATH

RUN mv jdk1.8.0_181 jdk && mv apache-tomcat-8.5.73 tomcat

ENV JAVA_HOME=/usr/local/jdk
ENV CLASS_PATH=$JAVA_HOME/lib:$JAVA_HOME/jre/lib
ENV PATH=$JAVA_HOME/bin:$PATH
ENV CATALINA_HOME /usr/local/tomcat

EXPOSE 8080  8443

CMD /usr/local/tomcat/bin/startup.sh && tail -F /usr/local/tomcat/bin/logs/catalina.out
```

