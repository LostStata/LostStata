FROM centos:7.2.1511

ADD jdk-8u181-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-8.5.73.tar.gz /usr/local/

ENV MYPATH /usr/local/
WORKDIR $MYPATH

RUN mv jdk1.8.0_181 jdk && mv apache-tomcat-8.5.73 tomcat

ENV JAVA_HOME=/usr/local/jdk
ENV CLASS_PATH=$JAVA_HOME/lib:$JAVA_HOME/jre/lib
ENV PATH=$JAVA_HOME/bin:$PATH
ENV CATALINA_HOME /usr/local/tomcat

EXPOSE 8080  8443

CMD /usr/local/tomcat/bin/startup.sh && tail -F /usr/local/tomcat/bin/logs/catalina.out