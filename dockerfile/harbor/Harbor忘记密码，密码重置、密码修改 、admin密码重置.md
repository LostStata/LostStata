### Harbor忘记密码，密码重置、密码修改 、admin密码重置

#### 一、找到harbor-db的容器，进入容器

``` shell
[root@localhost harbor]# docker ps
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS                    PORTS                       NAMES
77cf26066822        goharbor/harbor-jobservice:v2.4.1    "/harbor/entrypoin..."   18 minutes ago      Up 18 minutes (healthy)                               harbor-jobservice
64b5ac52ed66        goharbor/nginx-photon:v2.4.1         "nginx -g 'daemon ..."   18 minutes ago      Up 18 minutes (healthy)   0.0.0.0:8082->8080/tcp      nginx
beb088f4dfc5        goharbor/harbor-core:v2.4.1          "/harbor/entrypoin..."   18 minutes ago      Up 18 minutes (healthy)                               harbor-core
ead6f70375c4        goharbor/redis-photon:v2.4.1         "redis-server /etc..."   18 minutes ago      Up 18 minutes (healthy)                               redis
a6b2ea24ff49        goharbor/harbor-db:v2.4.1            "/docker-entrypoin..."   18 minutes ago      Up 18 minutes (healthy)                               harbor-db
a24d456d080d        goharbor/harbor-portal:v2.4.1        "nginx -g 'daemon ..."   18 minutes ago      Up 18 minutes (healthy)                               harbor-portal
8d9ecdc2e25d        goharbor/registry-photon:v2.4.1      "/home/harbor/entr..."   18 minutes ago      Up 18 minutes (healthy)                               registry
9fca641428e6        goharbor/harbor-registryctl:v2.4.1   "/home/harbor/star..."   18 minutes ago      Up 18 minutes (healthy)                               registryctl
0d8486117391        goharbor/harbor-log:v2.4.1           "/bin/sh -c /usr/l..."   18 minutes ago      Up 18 minutes (healthy)   127.0.0.1:1514->10514/tcp   harbor-log


[root@localhost harbor]# docker exec -it a6b2ea24ff49 /bin/bash
postgres [ / ]$ 

```



#### 二、进入postgresql命令行

psql -h postgresql -d postgres -U postgres

``` sehll
postgres [ / ]$ psql -h postgresql -d postgres -U postgres
Password for user postgres: 
psql (13.5)
Type "help" for help.

postgres=# 

```

默认密码root123



#### 三、切换到harbor所在的数据库

\c registry

``` shell
postgres=# \c registry
You are now connected to database "registry" as user "postgres".
```



#### 四、查看harbor_user表

select * from harbor_user;

```shell
registry=# select * from harbor_user;
 user_id | username  | email |             password             |    realname    |    comment     | deleted | reset_uuid |               salt               | sysadmin_flag |       creatio
n_time        |        update_time         | password_version 
---------+-----------+-------+----------------------------------+----------------+----------------+---------+------------+----------------------------------+---------------+--------------
--------------+----------------------------+------------------
       2 | anonymous |       |                                  | anonymous user | anonymous user | t       |            |                                  | f             | 2022-03-02 09
:50:01.274432 | 2022-03-02 09:50:01.378737 | sha1
       1 | admin     |       | 2848dc68148add8626b1c45b2a83546d | system admin   | admin user     | f       |            | JTsJakoSdKXsdulCTUgdZMfFgqMUs4qL | t             | 2022-03-02 09
:50:01.274432 | 2022-03-02 10:04:26.87987  | sha256
(2 rows)

```

#### 五、重置密码

修改admin的密码，修改为初始化密码Harbor12345 ，修改好了之后登录web ui上再修改



``` shell
registry=# update harbor_user set password='2848dc68148add8626b1c45b2a83546d', salt='JTsJakoSdKXsdulCTUgdZMfFgqMUs4qL' where username='admin';
UPDATE 1
```

