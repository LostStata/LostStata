# cfssl工具生成HTTPS证书来搭建Harbor镜像仓库

cfssl工具生成HTTPS证书来搭建Harbor镜像仓库

 离线部署Harbor(超详细）

```
Harbor官方网站：http://vmware.github.io/harbor/
Harbor下载地址：https://github.com/vmware/harbor/
```

harbor的二进制包同时提供online（在线安装）和offline（离线安装）版本

Harbor安装方式

离线安装：资源包带offline，文件内含安装需要的一些镜像文件，容量就比较大，约600M,适用于生产环境使用。

在线安装：资源包带online，安装时需要的镜像等资源需要从网上下载，所有要保证主机可以联通互联网。

适用于搭建私有的镜像仓库，用于学习和研究使用。

源码安装：需要了解Harbor底层原理和实现方式的，可选择源码安装的方式，适合对Harbor做开发的环境搭建。

## 一、安装docker、docker-compose

``` shell
# yum install -y docker-ce-19.03.8-3.el7.x86_64.rpm
# systemctl start docker.service
# systemctl enable  docker.service
# mv  docker-compose-Linux-x86_64  /usr/local/bin/docker-compose
# chmod +x   /usr/local/bin/docker-compose
# docker-compose -v
docker-compose version 1.29.2, build 5becea4c
```

## 二、配置内核参数

``` shell
# cat > /etc/sysctl.conf << EOF
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl -p
```

> **net.ipv4.ip_forward=1** ：开启路由转发，不配置该参数，当主机重启后，服务状态正常，却无法访问到服务器。

## 三、使用cfssl工具配置Harbor证书

> 默认情况下，Harbor 不附带证书。可以在没有安全性的情况下部署 Harbor，以便您可以通过 HTTP 连接到它。在生产环境中，始终使用 HTTPS。如果启用 Content Trust with Notary 以正确签署所有图像，则必须使用 HTTPS。 
>
> 要配置 HTTPS，必须创建 SSL 证书。您可以使用受信任的第三方 CA 签署的证书，也可以使用自签名证书。本节介绍如何使用cfssl创建 CA，以及如何使用您的 CA 签署服务器证书和客户端证书。
>
> 官方使用的是openssl工具来生成证书。

### 1、下载并安装cfssl工具

``` shell
# wget https://github.com/cloudflare/cfssl/releases/download/v1.6.0/cfssl_1.6.0_linux_amd64  -O   /usr/local/bin/cfssl
# wget https://github.com/cloudflare/cfssl/releases/download/v1.6.0/cfssljson_1.6.0_linux_amd64 -O  /usr/local/bin/cfssljson
# wget https://github.com/cloudflare/cfssl/releases/download/v1.6.0/cfssl-certinfo_1.6.0_linux_amd64  -O  /usr/local/bin/cfssl-certinfo 
# chmod +x  /usr/local/bin/cfssl*                  #给这几个工具执行权限
```

> **cfssljson**：将从cfssl和multirootca等获得的json格式的输出转化为证书格式的文件（证书，密钥，CSR和bundle）进行存储；
>
> **cfssl-certinfo**：可显示CSR或证书文件的详细信息；可用于证书校验。

### 2、生成证书颁发机构证书

（1）生成并修改CA默认配置文件

``` shell
# cfssl print-defaults  config > ca-config.json            #生成默认配置文件
# vim ca-config.json
{
    "signing": {
        "default": {
            "expiry": "87600h"
        },
        "profiles": {
            "harbor": {
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
```

> default.expiry：默认证书有效期（单位：h）
>
> profiles.harbor：为服务使用该配置文件颁发证书的配置模块；
>
> signing：签署，表示该证书可用于签名其它证书；生成的 ca.pem 证书中 CA=TRUE；
>
> key encipherment：密钥加密；
>
> profiles：指定了不同角色的配置信息；可以定义多个 profiles，分别指定不同的过期时间、使用场景等参数；后续在签名证书时使用某个 profile。
>
> server auth：服务器身份验证；表示 client 可以用该 CA 对 server 提供的证书进行验证；
>
> client auth：客户端身份验证；表示 server 可以用该 CA 对 client 提供的证书进行验证；

（2）生成并修改默认csr请求文件

``` shell
# cfssl  print-defaults csr  > ca-csr.json
# vim ca-csr.json
{
    "CN": "harbor",
    "hosts": [
        "192.168.254.128"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "Beijing",
            "L": "Beijing"
        }
    ]
}
```

> hosts：包含的授权范围，不在此范围的的节点或者服务使用此证书就会报证书不匹配错误，证书如果不包含可能会出现无法连接的情况；
>
> Key: 指定使用的加密算法，一般使用rsa非对称加密算法（algo:rsa；size:2048）
>
> CN：Common Name，kube-apiserver 从证书中提取该字段作为请求的用户名 (User Name)；浏览器使用该字段验证网站是否合法；CN是域名，也就是你现在使用什么域名就写什么域名
>
> O：Organization，kube-apiserver 从证书中提取该字段作为请求用户所属的组 (Group)；

（3）初始化CA

``` shell
# cfssl  gencert  -initca  ca-csr.json  |  cfssljson  -bare   ca
2022/01/18 15:45:13 [INFO] generating a new CA key and certificate from CSR
2022/01/18 15:45:13 [INFO] generate received request
2022/01/18 15:45:13 [INFO] received CSR
2022/01/18 15:45:13 [INFO] generating key: rsa-2048
2022/01/18 15:45:13 [INFO] encoded CSR
2022/01/18 15:45:13 [INFO] signed certificate with serial number 569300079090788296899255431042064535929535986450
# ls 
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem
```

可以看到，当前目录下新生成了ca.csr、ca-key.pem、ca.pem这３个文件。

ca-key.pem、ca.pem这两个是CA相关的证书，通过这个CA来签署服务端证书。

### 3、生成服务器证书

（1）创建并修改服务器证书请求文件

``` shell
# cfssl  print-defaults csr >  harbor-csr.json
# vim　 harbor-csr.json
{
    "CN": "muli.harbor.json",
    "hosts": [],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "Beijing",
            "L": "Beijing"
        }
    ]
}
```

（2）使用请求文件根据CA配置颁发证书

切换到证书所在目录，执行以下命令

``` shell
# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem \
-config=ca-config.json \
-profile=harbor  harbor-csr.json | cfssljson -bare  harbor
# ls
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem  harbor.csr  harbor-csr.json  harbor-key.pem  harbor.pem
# cp harbor.pem harbor-key.pem  /etc/docker/CA/
```

> ***\*-config\*******\*：\****指定CA证书机构的配置文件；
>
> ***\*-profile\*******\*：\****指定使用CA配置文件中的哪个模块（此处harbor对应配置文件中的harbor）；
>
> **harbor.pem****：**harbor服务的数字证书
>
> **harbor-key.pem**：harbor服务的私钥

### 4、harbor配置文件中指定使用证书

``` shell
# vim  harbor.yml
https:
  port: 443
  certificate: /etc/docker/CA/harbor.pem
  private_key: /etc/docker/CA/harbor-key.pem
```

## 四、部署Harbor（离线包）

###  4.1、下载并解压安装包

``` shell
在线下载地址：https://goharbor.io/
# yum -y install wget
# wget https://github.com/goharbor/harbor/releases/download/v2.4.1/harbor-offline-installer-v2.4.1.tgz
离线下载后上传
# rz  harbor-offline-installer-v2.3.1.tgz     //上传harbor安装包
# tar zxvf harbor-offline-installer-v2.3.1.tgz
# cd  harbor
# pwd
/dockerfile/harbor/harbor
# ls
common.sh  harbor.v2.3.1.tar.gz  harbor.yml.tmpl  install.sh  LICENSE  prepare
```

harbor.v2.3.1.tar.gz：镜像包，运行install.sh脚本会自动导入该镜像；

### 4.2、使用模板文件创建配置文件

``` shell
# cp harbor.yml.tmpl harbor.yml.tmpl.bak
# cp harbor.yml.tmpl harbor.yml
# vim harbor.yml
hostname: 192.168.254.128           #域名或IP
http:
  port: 80
https:
  port: 443
  certificate: /your/certificate/path
  private_key: /your/private/key/path
#Harbor管理员（admin）的密码，初次不可修改
harbor_admin_password: Harbor12345
database:
  password: root123              #数据库密码
  max_idle_conns: 100
  max_open_conns: 900
data_volume: /app/harbor/data         #容器存储卷数据存放位置
 
trivy:                   #配置Trivy扫描仪
  ignore_unfixed: true
  skip_update: false
  insecure: false
jobservice:                  #配置镜像上传/下载并发量
  max_job_workers: 10
notification:
  webhook_job_max_retry: 10
chart:
  absolute_url: disabled
log:
  level: info
  local:
    rotate_count: 50
    rotate_size: 200M
    location: /var/log/harbor
_version: 2.3.1
proxy:
  http_proxy:
  https_proxy:
  no_proxy:
  components:
    - core
    - jobservice
    - trivy
```

使用我自己的配置文件，需要配置ip地址、端口、证书地址、数据库存储地址等

```shell
# cat harbor.yml
hostname: 192.168.254.128
http:
  port: 8082
https:
  port: 443
  certificate: /etc/docker/CA/harbor.pem
  private_key: /etc/docker/CA/harbor-key.pem
harbor_admin_password: Harbor12345
database:
  password: root123
  max_idle_conns: 100
  max_open_conns: 900
data_volume: /dockerfile/harbor/data
trivy:
  ignore_unfixed: true
  skip_update: false
  insecure: false
jobservice:
  max_job_workers: 10
notification:
  webhook_job_max_retry: 10
chart:
  absolute_url: disabled
log:
  level: info
  local:
    rotate_count: 50
    rotate_size: 200M
    location: /var/log/harbor
_version: 2.3.1
proxy:
  http_proxy:
  https_proxy:
  no_proxy:
  components:
    - core
    - jobservice
    - trivy
```

配置文件的详细配置见官网（[Harbor docs | Configure the Harbor YML File](https://goharbor.io/docs/2.3.0/install-config/configure-yml-file/)）

### 4.3、运行prepare脚本以启用HTTPS

确保配置文件中启用https模式。并配置好证书

``` shell
# ./prepare
```

### 4.4、执行install.sh脚本安装harbor

在线安装方式会从网上自动下载需要的镜像，离线安装是自动将harbor.v2.3.1.tar.gz镜像包导入到本地镜像。

``` shell
# ./install.sh
```

![](assets\install.jpg)

### 4.5、验证服务是否正常

``` shell
[root@localhost harbor]# pwd
/dockerfile/harbor/harbor
[root@localhost harbor]# docker-compose ps  #需要到harbor安装目录执行
      Name                     Command                  State                           Ports                    
-----------------------------------------------------------------------------------------------------------------
harbor-core         /harbor/entrypoint.sh            Up (healthy)                                                
harbor-db           /docker-entrypoint.sh 96 13      Up (healthy)                                                
harbor-jobservice   /harbor/entrypoint.sh            Up (healthy)                                                
harbor-log          /bin/sh -c /usr/local/bin/ ...   Up (healthy)   127.0.0.1:1514->10514/tcp                    
harbor-portal       nginx -g daemon off;             Up (healthy)                                                
nginx               nginx -g daemon off;             Up (healthy)   0.0.0.0:8082->8080/tcp, 0.0.0.0:443->8443/tcp
redis               redis-server /etc/redis.conf     Up (healthy)                                                
registry            /home/harbor/entrypoint.sh       Up (healthy)                                                
registryctl         /home/harbor/start.sh            Up (healthy)
```

当stata（状态）为up时，表示容器启动正常。

排错：harbor-db状态显示为restarting，需要删除数据库里的数据，重新执行./install.sh

## 五、harbor启停

### 5.1、停止并删除现有实例

数据保留在文件系统中，因此不会丢失任何数据。

``` shell
# docker-compose down -v
```

![](assets\删除harbor.jpg)

### 5.2、重启Harbor

``` shell
# docker-compose up -d
```

![](assets\重启harbor.jpg)

## 六、登录Harbor管理控制台

> 浏览器输入访问地址：
>
> https模式：https://192.168.254.128:443
>
> http 模式： http://192.168.254.128:8082
>
> 用户：admin
>
> 密码：harbor.cfg文件中harbor_admin_password的值。

![](assets\harborlogin.jpg)

![sys界面](assets\harbor界面.jpg)

## 七、配置docker使用Harbor镜像仓库

> 需要在docker配置文件中指定harbor地址，才可以从命令行登录harbor。

``` shell
[root@localhost harbor]# cat /etc/docker/daemon.json
{
 "registry-mirrors": ["https://tyfieo0x.mirror.aliyuncs.com"],
 "insecure-registries":["192.168.254.128:443"]
}
```

命令行连接/退出Harbor

``` shell
# docker login -u 用户名 -p 密码 192.168.254.128:443
[root@localhost harbor]# docker login -u admin -p Harbor12345 192.168.254.128:443
Login Succeeded
# docker  logout 192.168.254.128:443
[root@localhost harbor]# docker logout 192.168.254.128:443
Removing login credentials for 192.168.254.128:443
```

# 备注：

————————————————
版权声明：本文为CSDN博主「帮我起个名」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_49496397/article/details/121176817





































