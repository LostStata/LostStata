1、安装docker配置环境

2、docker搭建zabbix
编写docker-compose.yml
###############
version: '3.5'
services:

  mysql: 
    image: mysql:5.7
    container_name: zabbix-mysql
    restart: unless-stopped
    ports:
      - 53306:3306
    command: [
      '--default-time-zone=+8:00'
    ]
    environment:
      TZ: Asia/Shanghai
      MYSQL_DATABASE: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: mypassword
      MYSQL_ROOT_PASSWORD: mypassword
    volumes:
      - /etc/localtime:/etc/localtime
      - ./mysql/data:/var/lib/mysql
    networks: 
      - zabbix

  zabbix-server:
    image: zabbix/zabbix-server-mysql:alpine-4.4-latest
    container_name: zabbix-server
    restart: unless-stopped
    ports:
      - 10051:10051
    depends_on:
      - mysql
    environment:
      DB_SERVER_HOST: mysql
      MYSQL_DATABASE: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: mypassword
      MYSQL_ROOT_PASSWORD: mypassword
    networks: 
      - zabbix

  zabbix-web:
    image: zabbix/zabbix-web-nginx-mysql:alpine-4.4-latest
    container_name: zabbix-web
    restart: unless-stopped
    ports:
      - 8080:8080
    depends_on:
      - zabbix-server
      - mysql
    environment:
      PHP_TZ: Asia/Shanghai
      DB_SERVER_HOST: mysql
      MYSQL_DATABASE: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: mypassword
      MYSQL_ROOT_PASSWORD: mypassword
    volumes:
      - ./fonts:/usr/share/zabbix/assets/fonts
    networks: 
      - zabbix

networks: 
  zabbix:
    driver: bridge
    ipam:
      config:
        - subnet:  192.168.20.0/24
################
3、启动服务
docker-compose up -d
Bash
4、登录
端口：8080
默认账号 Admin
初始密码 zabbix
5、Web前端中文化
在后台配置中文语言：进入“User”设置项，在“Language”处修改为“Chinese（zh_CN）”，然后点击“Update”更新。

6、配置服务端
配置 – 主机 – Zabbix server
agent代理程序的接口：本机内网IP（例如 192.168.0.5，注意不能是回环地址），端口 10050
查看 agent 日志：tail -f /var/log/zabbix/zabbix_agentd.log

7、图表中文字体
复制字体文件到 docer-compose.yml 目录下的 fonts 目录下，文件名为 DejaVuSans.ttf 。

8、安装 agent
apt install zabbix-agent
service zabbix-agent start
netstat -ntpl | grep zabbix

9、配置 agent
Server=192.168.20.0/24 允许来自该网域的管理连接
Hostname=Zabbix server
