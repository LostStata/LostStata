##### registry仓库

关于SSL证书，自制证书：可通过linux的OpenSSL工具创建SSL证书，不过这种无法使用

###### 收费证书：各种云主机提供商

###### 创建映射目录
``` shell
mkdir -p /data/docker.registry
mkdir -p /data/docker.registry/etc/cert.d
mkdir -p /data/docker.registry/var/lib/registry
```



###### 复制SSL证书，域名是registry.tongfu.net
复制registry.crt文件和registry.key文件到/data/docker.registry/etc/cert.d下面。

``` shell
cp registry.crt /data/docker.registry/etc/cert.d/registry.crt
cp registry.key /data/docker.registry/etc/cert.d/registry.key
```



###### 启动私有仓库容器

重新创建私有仓库的容器，配置SSL证书参数

``` shell 
docker run -tid \
--name registry \
-h registry \
-p 5000:5000 \
--memory 512m \
--memory-swap -1 \
--restart always \
-v /data/docker.registry/etc/cert.d:/etc/cert.d \
-v /data/docker.registry/var/lib/registry:/var/lib/registry \
-e REGISTRY_HTTP_TLS_CERTIFICATE=/etc/cert.d/registry.crt \
-e REGISTRY_HTTP_TLS_KEY=/etc/cert.d/registry.key \
registry
```



###### 添加本地DNS
在/etc/hosts里面添加域名解析。

``` shell
127.0.0.1 registry.tongfu.net
```



###### 测试

``` shell
curl 'https://registry.tongfu.net:5000/v2/_catalog'
```



###### 给nginx镜像打标签

``` shell
docker tag nginx:latest registry.tongfu.net:5000/nginx
```



###### 提交带标签的nginx镜像到私有仓库

``` shell
docker push registry.tongfu.net:5000/nginx
```



###### 查看仓库容器

``` shell
curl 'https://registry.tongfu.net:5000/v2/_catalog'
```



##使用自制证书，查看的时候需要加-k参数，只能本地使用不建议
##删除带标签的镜像，带标签的镜像IMAGE ID跟原始的镜像是一致的，不能直接使用IMAGE ID删除，需要使用完整的镜像名加版本号。

简单使用OpenSSL创建自制证书

``` shell
[root@spark32 ~]# mkdir -p /opt/docker/registry/certs
[root@spark32 ~]# openssl req -newkey rsa:4096 -nodes -sha256 -keyout /opt/docker/registry/certs/domain.key -x509 -days 365 -out /opt/docker/registry/certs/domain.crt
Generating a 4096 bit RSA private key
...
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:JiangSu
Locality Name (eg, city) [Default City]:NanJing
Organization Name (eg, company) [Default Company Ltd]:wisedu
Organizational Unit Name (eg, section) []:edu
Common Name (eg, your name or your server's 
```

