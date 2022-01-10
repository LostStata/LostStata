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