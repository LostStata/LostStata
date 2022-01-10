docker容器跨网络操作

```
[root@VM-16-4-centos zabbix]# docker network ls
NETWORK ID       NAME             DRIVER     SCOPE
7032404e9d6d     bridge            bridge    local
30b881f029cd      emp              bridge    local
80f87f3cd20d      host             host      local
68b0cc68670a      none             null      local
7bc34fe8c2e8   zabbix_zabbix       bridge    local
```

```
[root@VM-16-4-centos zabbix]# docker network connect zabbix_zabbix nginx_emp
```

```
[root@VM-16-4-centos zabbix]# docker exec -it nginx_emp ping zabbix-server

PING zabbix-server (192.168.20.3) 56(84) bytes of data.
64 bytes from zabbix-server.zabbix_zabbix (192.168.20.3): icmp_seq=1 ttl=64 time=0.080 ms
64 bytes from zabbix-server.zabbix_zabbix (192.168.20.3): icmp_seq=2 ttl=64 time=0.047 ms
64 bytes from zabbix-server.zabbix_zabbix (192.168.20.3): icmp_seq=3 ttl=64 time=0.047 ms
^C
--- zabbix-server ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.047/0.058/0.080/0.015 ms
```

docker中nginx_emp容器在emp网段，zabbix-server在zabbix_zabbix网络
使用docker  network  connect  网络段  容器名
之后发现容器通了











