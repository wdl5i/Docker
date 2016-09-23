# 使用Dockerfile创建ssh镜像 #
## 编写run.sh ##
在宿主机上编写run.sh
<pre>
#!/bin/bash
/usr/sbin/sshd -D
</pre>
## 生成SSH密钥对
在宿主机上生成ssh密钥对，并创建authorized_keys文件：  
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub > authorized_keys
## 编写Dockerfile ##
<pre>
#设置基础镜像
FROM ubuntu:vim

#设备创建和维护人信息
MAINTAINER wangdonglin wdl5i@163.com

#更新ubuntu的源为国内163的源
RUN echo "deb http://mirrors.163.com/ubuntu/ trusty main restricted universe multiverse" > /etc/apt/sources.list
RUN echo "deb http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb http://mirrors.163.com/ubuntu/ trusty-updates main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb http://mirrors.163.com/ubuntu/ trusty-proposed main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb http://mirrors.163.com/ubuntu/ trusty-backports main restricted universe multiverse" >> /etc/apt/sources.list
RUN apt-get update
RUN apt-get upgrade -y

#安装sshd
RUN apt-get install -y openssh-server
RUN mkdir -p /var/run/sshd
RUN mkdir -p /root/.ssh

#取消pam限制
RUN sed -ri 's/session    required     pam_loginuid.so/#session    required     pam_loginuid.so/g' /etc/pam.d/sshd

#复制配置文件到相应位置,并赋予脚本可执行权限
ADD authorized_keys /root/.ssh/authorized_keys
ADD run.sh /run.sh
RUN chmod 755 /run.sh

#开放端口
EXPOSE 22

#设置自启动命令
CMD ["/run.sh"]
</pre>
## 创建镜像 ##
在sshd_ubuntu目录下，创建sshd镜像，镜像名为sshd:dockerfile
<pre>
cd sshd_ubuntu
docker build -t sshd:dockerfile .
</pre>
命令执行完成，当看到有Successful build XXX字样，代表镜像创建成功
## 测试镜像 ##
<pre>
docker run -d -p 10022:22 sshd:dockerfile
[root@192 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                   NAMES
045a1a8f5308        sshd:dockerfile     "/run.sh"           12 minutes ago      Up 12 minutes       0.0.0.0:10022->22/tcp   determined_meitner 
ssh 127.0.0.1 -p 10022
The authenticity of host '[127.0.0.1]:10022 ([127.0.0.1]:10022)' can't be established.
RSA key fingerprint is dc:b4:19:eb:73:73:45:5b:10:6f:30:fa:3f:ae:d6:1a.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[127.0.0.1]:10022' (RSA) to the list of known hosts.
</pre>
ssh已经成功连接  
# 使用Dockerfile创建java镜像 #
## 编写Dockerfile ##
<pre>
#使用ubuntu_sshd镜像
FROM sshd:dockerfile

#刷新包缓存 并且 安装wget,tar工具  
RUN apt-get update && apt-get install -y wget  && apt-get install -y tar

WORKDIR /usr/local

RUN wget --no-cookies --no-check-certificate --header "Cookie:gpw_e24=http%3a%2f%2fwww.oracle.com%2ftechnetwork%2fjava%2fjavase%2fdownloads%2fjdk7-downloads-1880260.html;oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/7u79-b15/jdk-7u79-linux-x64.tar.gz  
RUN tar -zxf jdk-7u79-linux-x64.tar.gz 

ENV JAVA_HOME /usr/local/jdk1.7.0_79  
ENV JRE_HOME $JAVA_HOME/jre  
ENV CLASSPATH .:$JAVA_HOME/lib:$JRE_HOME/lib  
ENV PATH $PATH:$JAVA_HOME/bin 
##################################以下软件基于ubuntu:java安装##################################
#安装 tomcat7  
FROM ubuntu:java
MAINTAINER wangdonglin wdl5i@163.com
WORKDIR /usr/local  
RUN wget http://mirror.bit.edu.cn/apache/tomcat/tomcat-7/v7.0.72/bin/apache-tomcat-7.0.72.tar.gz
RUN tar xvf apache-tomcat-7.0.62.tar.gz
 
ENV CATALINA_HOME /usr/local/apache-tomcat-7.0.62

EXPOSE 8080  
  
CMD [ "/usr/local/apache-tomcat-7.0.62/bin/catalina.sh", "run" ]  

#安装mysql
RUN apt-get install -y mysql-server mysql
RUN /etc/init.d/mysqld start &&\  
    mysql -e "grant all privileges on *.* to 'root'@'%' identified by '48STX2X';"&&\  
    mysql -e "grant all privileges on *.* to 'root'@'localhost' identified by '48STX2X';"&&\  
    mysql -u root -48STX2X -e "show databases;"  
   
EXPOSE 3306  
   
CMD ["/usr/bin/mysqld_safe"]

#安装Redis  
RUN wget http://download.redis.io/releases/redis-3.0.3.tar.gz
RUN tar –zxf redis-3.0.3.tar.gz –C /usr/local/   --strip-components=1
 
RUN apt-get install –y gcc libc6-dev make 
RUN make –C /usr/local/redis-3.0.3
RUN make –C /usr/local/redis-3.0.3 install
RUN rm redis-3.0.3.tar.gz
RUN rm –r /usr/local/redis-3.0.3
 
EXPOSE 6379

</pre>

**是使最后sh文件执行，必须是以docker run -d的形式启动，也有可能是CMD和ENTRYPOINT的关系**