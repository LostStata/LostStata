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

##### 2、安装Postfix以发送电子邮件通知

```shell
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix

#下载完gitlab-ce后，配置postfix
vim打开gitlab的配置文件：/etc/gitlab/gitlab.rb，新增以下内容

smtp_addressQQ邮箱服务器是smtp.qq.com
smtp_port端口465 （注意，不要用25端口）
smtp_user_name 配置自己的QQ号
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.qq.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "2833xxx@qq.com"  # 你自己QQ号
gitlab_rails['smtp_password'] = "*************"             # QQ授权码
gitlab_rails['smtp_domain'] = "smtp.qq.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
gitlab_rails['gitlab_email_from'] = '2833xxx@qq.com' #你自己的qq号

#配置完成后即可使用邮箱注册账号，创建账号时需使用邮箱验证激活
#测试邮箱
执行 gitlab-rails console进入控制台交互界面, 然后在控制台提示符后输入下面内容发送一封测试邮件，测试完成后exit()退出。

gitlab-rails console
Notify.test_email('yoyo_你自己随便邮箱@qq.com', '邮件标题_test', '邮件正文_test').deliver_now

[root@yoyo gitlab]# gitlab-rails console
Loading production environment (Rails 4.2.8)
irb(main):001:0> Notify.test_email('yoyo_******@qq.com', '邮件标题_test', '邮件正文_test').deliver_now 

Notify#test_email: processed outbound mail in 1.2ms

Sent mail to yoyo_******@qq.com(1375.0ms)
Date: Fri, 18 Jan 2019 13:58:24 +0800
From: GitLab <2833xxx@qq.com>
Reply-To: GitLab <noreply@47.104.190.48>
To: yoyo_******@qq.com
Message-ID: <5c416b00e10ef_3e8f3fe6bd9db11817659@yoyo.mail>
Subject: =?UTF-8?Q?=E9=82=AE=E7=AE=B1=E4=B8=BB=E9=A2=98=5Ftest?=
Mime-Version: 1.0
Content-Type: text/html;
 charset=UTF-8
Content-Transfer-Encoding: quoted-printable
Auto-Submitted: auto-generated
X-Auto-Response-Suppress: All

<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN" "http://www=
.w3.org/TR/REC-html40/loose.dtd">
<html><body><p>=E9=82=AE=E7=AE=B1=E6=AD=A3=E6=96=87_test</p></body></html=
>

=> #<Mail::Message:70259829672900, Multipart: false, Headers: <Date: Fri, 18 Jan 2019 13:58:24 +0800>, 
<From: GitLab <2833xxx@qq.com>>, <Reply-To: GitLab <noreply@47.104.190.48>>, 
<To: yoyo_******@qq.com>, <Message-ID: <5c416b00e10ef_3e8f3fe6bd9db11817659@yoyo.mail>>, 
<Subject: 邮箱主题_test>, <Mime-Version: 1.0>, <Content-Type: text/html; charset=UTF-8>, 
<Content-Transfer-Encoding: quoted-printable>, 
<Auto-Submitted: auto-generated>, 
<X-Auto-Response-Suppress: All>>
irb(main):006:0> exit()  # 退出
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



