##### 1、下载所需的源码包

```
wget http://nginx.org/download/nginx-1.14.0.tar.gz
```

##### 2、编写完成dockerfile后创建镜像

```
docker bulid  -t  nginx1.14.0 .
```

##### 3、创建挂载nginx所需的文件，复制原始nginx.conf到挂载目录

```
mkdir  /data/nginx/conf
tar  -xf  nginx-1.14.0.tar.gz  -C  /data/nginx
cp  -r  nginx-1.14.0/conf/nginx.conf  /data/nginx/conf/
```

##### 4、修改nginx配置文件，添加所需的修改项，创建nginx容器

```
使用创建的emp网络

docker  run  -d  -p  80:80  -p  443:443  --net  emp  --ip  172.16.125.20  --name  nginx_emp  --restart=always   -v  /data/nginx/conf/nginx.conf:/opt/nginx/conf/nginx.conf    nginx:1.14.0
```

##### 注释：

tomcat修改挂载的文件，容器内文件不能修改成功，原因是：挂载进docker的文件，实际上是docker记住了一个iNode，他可以通过这个iNode找到block，也就是实际的文件信息.如果是用 > 追加重定向写入文件，是可以同步到docker的,但是如果是rm 重命名的，文件的iNode就改变了，但是docker中的iNode还是指向了之前的磁盘位置，所以文件没有改变
同样的还有vim ，vim 文件的时候是基于现有的文件copy了一份，同级目录下会有一个，开头swp结尾的文件，当你保存退出的时候，vim 会删掉源文件，将这个文件重命名为源文件的名字，iNode自然也就改变了。
建议固定容器ip，ip发生改变后，需要修改容器内部nginx的配置ip地址

##### 备注：

如需更改其他文件，在容器挂载的时候挂载出来，进行配置

###################

##### 查看容器ip

```
docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器id
```

###################

###################

```
[root@VM-16-4-centos tomcat01]# echo "where are you from" > webapps/ROOT/index.html
[root@VM-16-4-centos tomcat01]# echo "hello world" > /data/tomcat/tomcat02/webapps/ROOT/index.html
[root@VM-16-4-centos tomcat01]# echo "my world" > /data/tomcat/tomcat03/webapps/ROOT/index.html
```



##### 编写nginx镜像文件，Dockerfile

```
FROM centos:7.2.1511

ENV TZ=Asia/Shanghai

RUN yum -y install epel* \
	yum -y install gcc openssl openssl-devel  pcre-devel zlib-devel

ADD nginx-1.14.0.tar.gz /opt/

WORKDIR /opt/nginx-1.14.0

RUN ./configure --prefix=/opt/nginx  --http-log-path=/opt/nginx/logs/access.log --error-log-path=/opt/nginx/logs/error.log --http-client-body-temp-path=/opt/nginx/client/  --http-proxy-temp-path=/opt/nginx/proxy/  --with-http_stub_status_module --with-file-aio --with-http_flv_module --with-http_gzip_static_module --with-stream --with-threads --user=www --group=www
RUN make && make install
RUN groupadd www && useradd -g www www
WORKDIR /opt/nginx
RUN rm -rf /opt/nginx-1.14.0

ENV NGINX_HOME=/opt/nginx
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/nginx/sbin

EXPOSE 80 443

CMD ["nginx", "-g", "daemon off;"]
```
