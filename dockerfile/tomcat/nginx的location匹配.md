### nginx的location匹配

``` 
location [=|~|~*|^~] /uri/ { … }
```

- =     严格匹配。如果请求匹配这个location，那么将停止搜索并立即处理此请求
- ~     区分大小写匹配(可用正则表达式)
- ~*    不区分大小写匹配(可用正则表达式)
- !~    区分大小写不匹配
- !~*   不区分大小写不匹配
- ^~    如果把这个前缀用于一个常规字符串,那么告诉nginx 如果路径匹配那么不测试正则表达式



### [tomcat版本号进行隐藏或者删除](https://www.cnblogs.com/coding8832/p/14472131.html)

1.修改之前默认报错页面信息会暴露出版本号

2.进入tomcat的lib目录找到catalina.jar文件

3.unzip catalina.jar之后会多出两个文件夹

4.进入org/apache/catalina/util 编辑配置文件ServerInfo.properties

5.修改为

server.info=Apache Tomcat

server.number=0.0.0.0

server.built=Nov 7 2016 20:05:27 UTC

6.将修改后的信息压缩回jar包

cd  /tomcat/lib

jar uvf catalina.jar org/apache/catalina/util/ServerInfo.properties

7.重启tomcat



### 设置tomcat为中文

打开tomcat/conf/logging.properties文件,定位java.util.logging.ConsoleHandler.encoding = GBK，修改UTF -8 为GBK