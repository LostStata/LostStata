使用docker搭建个人blog

解压blog.tar.gz，进入到blog。

1、创建镜像
nginx采用nginx:1.18.0，mysql采用mysql:5.7，php采用tsund/php:7.2.3-fpm。
使用Dockerfile创建tsund/php:7.2.3-fpm镜像
docker build -t tsund/php:7.2.3-fpm .

2、容器编排
进入blog，执行docker-compose up -d进行

blog文件夹，目录如下
.
├── docker-compose.yml      Docker Compose 配置文件
├── mysql                             mysql 持久化目录
├── mysql.env                       mysql 配置信息
├── nginx                              nginx 配置文件的持久化目录
├── ssl                                   ssl 证书目录
└── typecho                          站点根目录

创建失败需要确认是否端口被占用，服务是否启动
nginx采用81端口、446端口，php采用9000，mysql用3306
服务服务打不开需要设置站点根目录的权限
设置index.php、install.php的权限设置为777，还需要设置/usr/uploads加执行权限
chmod 777 index.php
chmod 777 install.php
chmod -R 777 /usr/uploads

站点根目录，安装typecho
wget http://typecho.org/downloads/1.1-17.10.30-release.tar.gz
tar xvf 1.1-17.10.30-release.tar.gz
mv build typecho

个人blog的版本升级

停止服务docker-compose stop

删除服务器上的旧文件
请在服务器上删除如下目录和文件
/admin/
/var/
/index.php
/install.php
注意:请千万不要删除/usr/目录，因为这个目录包含了你的主题，插件和上传的文件，它无需被升级
上传新文件，请把你下载的压缩文件解压后，上传以上已经删除的文件和目录，这实际上是执行了一次覆盖操作，
让我再来重复一遍需要上传的目录和文件
/admin/
/var/
/index.php
/install.php

当你没有进行下面的步骤时，访问前台页面可能回出现错误提示，请不要管他们，直接访问你的 admin 页面，按提示完成升级即可恢复正常
用一个具有管理员权限的用户登录后台，系统会提示检测到新版本需要升级，点击完成升级按钮即可完成升级
如果在升级完成后，进入首页出现 500 或其他错误，请进入 admin 页面禁用所有的插件，并启用默认模板。如果正常，请逐步排查插件或模板存在的问题。
