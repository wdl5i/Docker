# 网管Dockerfile #
nmp:  
<pre>
FROM centos:latest
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
ADD server.xml /usr/local/apache-tomcat-7.0.68/conf/
ADD setenv.sh /usr/local/apache-tomcat-7.0.68/bin/
#RUN tar xvf apache-tomcat-7.0.68.tar.gz
ENV CATALINA_HOME /usr/local/apache-tomcat-7.0.68  
EXPOSE 8080  
RUN echo "/usr/local/apache-tomcat-7.0.68/bin/startup.sh" >> /init.sh
#CMD [ "/usr/local/apache-tomcat-7.0.68/bin/catalina.sh", "run" ] 

#setup redis
ADD redis-3.0.3.bin.centos.tar.gz .
#RUN tar xvf redis-3.0.3.bin.centos.tar.gz
RUN cp /usr/local/redis-3.0.3/redis.conf /etc/redis.conf
EXPOSE 6379
RUN mkdir /var/log/redis
RUN echo "/usr/local/redis-3.0.3/bin/redis-server /etc/redis.conf" >> /init.sh
#CMD ["sh", "/init.sh"]

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

# 容器需要开放MySQL 3306端口
EXPOSE 3306
</pre>

tools:
<pre>
# 选择一个已有的os镜像作为基础  
FROM centos:nmp
   
# 镜像的作者  
MAINTAINER donglin.wang "dl.wang-ht@haige.com"  
   
# 安装openssh-server和sudo软件包，并且将sshd的UsePAM参数设置成no  
RUN yum install -y wget vim curl net-tools openssh-server  
#RUN sed -i 's/UsePAM yes/UsePAM no/g' /etc/ssh/sshd_config  
RUN sed -ri 's/session    required     pam_loginuid.so/#session  required  pam_loginuid.so/g' /etc/pam.d/sshd  
RUN echo "root:123456" | chpasswd  
   
# 下面这两句比较特殊，在centos6上必须要有，否则创建出来的容器sshd不能登录  
RUN ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key  
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key  
RUN ssh-keygen -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key 
 
# 启动sshd服务并且暴露22端口  
RUN mkdir /var/run/sshd  
EXPOSE 22
RUN echo "/usr/sbin/sshd -D" >> /init.sh

#RUN echo "while true; do bash;  done" >> /init.sh
RUN chmod +x /init.sh

ENTRYPOINT ["/bin/sh", "-c", "/init.sh"]

</pre>