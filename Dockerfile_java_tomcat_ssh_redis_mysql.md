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
RUN sed -ri 's/session    required     
pam_loginuid.so/#session    required     
pam_loginuid.so/g' /etc/pam.d/sshd

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
# 使用Dockerfile创建java/tomcat/redis镜像 #
## 编写Dockerfile ##
<pre>
FROM centos:sshd
MAINTAINER wangdonglin wdl5i@163.com
WORKDIR /usr/local

#setup java
ADD ./jdk-7u79-linux-x64.tar.gz .
#RUN tar -zxf jdk-7u79-linux-x64.tar.gz
ENV JAVA_HOME /usr/local/jdk1.7.0_79
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH .:$JAVA_HOME/lib:$JRE_HOME/lib
ENV PATH $PATH:$JAVA_HOME/bin

#setup tomcat
ADD apache-tomcat-7.0.68.tar.gz .
#RUN tar xvf apache-tomcat-7.0.68.tar.gz
ENV CATALINA_HOME /usr/local/apache-tomcat-7.0.68
EXPOSE 8080
#CMD [ "/usr/local/apache-tomcat-7.0.68/bin/catalina.sh", "run" ]
RUN echo "/usr/local/apache-tomcat-7.0.68/bin/startup.sh" >> /init.sh

#setup redis
ADD redis-3.0.3.bin.centos.tar.gz .
#RUN tar xvf redis-3.0.3.bin.centos.tar.gz
RUN cp /usr/local/redis-3.0.3/redis.conf /etc/redis.conf
EXPOSE 6379
RUN mkdir /var/log/redis
RUN echo "/usr/local/redis-3.0.3/bin/redis-server /etc/redis.conf" >> /init.sh
RUN echo "while true; do bash;  done" >> /init.sh
RUN chmod +x /init.sh
#CMD ["sh", "/init.sh"]
ENTRYPOINT  ["/bin/sh", "-c", "/init.sh"]

#setup mysql
ADD mysql-5.6.19-linux-glibc2.5-x86_64.tar.gz .
ADD 01.init.sql ./01.init.sql
RUN mv mysql-5.6.19-linux-glibc2.5-x86_64 mysql

# 添加测试用户mysql，密码mysql，并且将此用户添加到sudoers里
RUN useradd mysql
RUN echo "mysql:mysql" | chpasswd
RUN echo "mysql   ALL=(ALL)       ALL" >> /etc/sudoers

# 设置Mysql安装目录的权限
RUN cd mysql && chown -R mysql:mysql ./

# 复制已经准备好的my.cnf文件到Docker容器
COPY my.cnf /etc/my.cnf
RUN chown mysql:mysql /etc/my.cnf

#setup depend
RUN yum -y install perl perl-devel autoconf libaio
# 初始化数据库
RUN cd mysql && ./scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data/
# 设置MySQL的环境变量，若读者有其他的环境变量需要设置，也可以在这里添加。
ENV MYSQL_HOME /usr/local/mysql

# （不推荐下面的路径直接建立在Docker虚拟机上，推荐使用volume挂载方式）
# 在宿主机上创建一个数据库目录存储Mysql的数据文件
# mkdir -p mysql/data

# VOLUME 选项是将本地的目录挂在到容器中　此处要注意：当你运行-v　＜hostdir>:<Containerdir> 时要确保目录内容相同否则会出现数据丢失
# 对应关系如下
# mysql:mysql/data
VOLUME ["/usr/local/mysql/data"]
RUN cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
RUN echo "/etc/init.d/mysqld start" >> /init.sh
RUN echo "/usr/local/mysql/bin/mysql --socket=/usr/local/mysql/data/mysql.sock -e \"grant all privileges on *.* to 'root'@'%' identified by '48STX2X';\"" >> /init.sh
RUN echo "/usr/local/mysql/bin/mysql --socket=/usr/local/mysql/data/mysql.sock -e \"grant all privileges on *.* to 'root'@'localhost' identified by '48STX2X';\"" >> /init.sh
RUN echo "/usr/local/mysql/bin/mysql --socket=/usr/local/mysql/data/mysql.sock -uroot -p48STX2X < /usr/local/01.init.sql" >> /init.sh
RUN echo "while true; do bash;  done" >> /init.sh
RUN chmod +x /init.sh

# 容器需要开放MySQL 3306端口
EXPOSE 3306
ENTRYPOINT  ["/bin/sh", "-c", "/init.sh"]
</pre>

**最好用ENTRYPOINT，不用CMD**
  
启动容器  
 `docker run -d -p 3306:3306 -p 80:8080 -p 6379:6379 centos:nmp bash` 
 `docker run -d --name nmp -p 3306:3306 -p 10022:22 -p 80:8080 -p 6379:6379 -v /opt/share:/usr/local/apache-tomcat-7.0.68/webapps centos:nmp bash`

