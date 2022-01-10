#### gitlab-ce的安装教程

##### 1、下载安装gitlab所需的环境

```shell
sudo yum install -y curl policycoreutils-python openssh-server perl
sudo systemctl enable sshd
sudo systemctl start sshd
#配置firewalld
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld
```

##### 2、安装Postfi以发送电子邮件通知

```shell
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```

##### 3、下载gitlab-ce安装包

上此网页上查询需要的版本信息

网址:https://packages.gitlab.com/app/gitlab/gitlab-ce/search

![下载界面](assets\20220109205727.jpg)

```shell
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
sudo yum install gitlab-ce-10.3.1-ce.0.el7.x86_64

```

##### 4、配置/etc/gitlab/gitlab.rb

```shell
vi /etc/gitlab/gitlab.rb
更改 https://gitlab.example.com 为您要访问极狐GitLab 实例的 URL
external_url 'http://192.168.254.133:82'
nginx['listen_port'] = 82 #配置默认启动的端口
```

##### 5、执行配置文件生效并启动gitlab-ce

```shell
gitlab-ctl reconfigure
```

```shell
gitlab-ctl restart
```

##### 6、访问页面

查看初始密码；除非您在安装过程中指定了自定义密码，否则将随机生成一个密码并存储在 /etc/gitlab/initial_root_password 文件中(出于安全原因，24 小时后，此文件会被第一次 gitlab-ctl reconfigure 自动删除，因此若使用随机密码登录，建议安装成功初始登录成功之后，立即修改初始密码）。使用此密码和用户名 root 登录。

```shell
[root@localhost ~]# cat  /etc/gitlab/initial_root_password
# WARNING: This value is valid only in the following conditions
#          1. If provided manually (either via `GITLAB_ROOT_PASSWORD` environment variable or via `gitlab_rails['initial_root_password']` setting in `gitlab.rb`, it was provided before database was seeded for the first time (usually, the first reconfigure run).
#          2. Password hasn't been changed manually, either via UI or via command line.
#
#          If the password shown here doesn't work, you must reset the admin password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.

Password: kfUCGfPXOEkYcYzYU+g6NC17FCNhd1JmIJ+DkOEfkPs=

# NOTE: This file will be automatically deleted in the first reconfigure run after 24 hours.
```

访问页面

![登录界面](assets\20220109205249.jpg)



