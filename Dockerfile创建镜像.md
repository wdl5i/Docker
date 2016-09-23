# 使用Dockerfile创建镜像 #
## 基本结构 ##
Dockerfile由一行行命令语句组成，支持以#为开始的注释行， Dockerfile分为四个部分，基础镜像信息，维护者信息，镜像操作指令，容器启动时指行指令。如：  
<pre>
# This dockerfile uses the ubuntu image
# VERSION 2 - EDITION 1
# Author: docker_user
# Command format: Instruction [arguments / command] ..
 
# Base image to use, this must be set as the first line
FROM ubuntu
 
# Maintainer: docker_user <docker_user at email.com> (@docker_user)
MAINTAINER docker_user docker_user@email.com
 
# Commands to update the image
RUN echo "deb http://archive.ubuntu.com/ubuntu/ raring main universe" >> /etc/apt/sources.list
RUN apt-get update && apt-get install -y nginx
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf
 
# Commands when creating a new container
CMD /usr/sbin/nginx
</pre>
## 指令 ##
1). FROM: 第一条指令必须为FROM， 指定镜像名称，格式为FROM imageId 或FROM imageId:tag，例如FROM ubuntu或FROM ubuntu:12.04  
2). MAINTAINER: 镜像作者 ，格式为 MAINTAINER name    
3). RUN:格式为 RUN <command> 或 RUN ["executable", "param1", "param2"]  
前者将在shell终端中运行命令，即 /bin/sh -c；后者则使用 exec 执行。指定使用其它终端可以通过第二种方式实现，例如 RUN ["/bin/bash", "-c", "echo hello"]。
每条 RUN 指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用\来换行。  
4). CMD:支持三种格式
　　1.CMD ["executable","param1","param2"] 使用 exec 执行，推荐方式；
　　2.CMD command param1 param2 在 /bin/sh 中执行，提供给需要交互的应用；
　　3.CMD ["param1","param2"] 提供给 ENTRYPOINT 的默认参数；
指定启动容器时执行的命令，每个Dockerfile只能有一条 CMD 命令。如果指定了多条命令，只有最后一条会被执行。如果用户启动容器时候指定了运行的命令，则会覆盖掉 CMD 指定的命令。  
5). EXPOSE:格式为 EXPOSE port [port...]。
告诉Docker服务端容器暴露的端口号，供互联系统使用。在启动容器时需要通过-P，Docker主机会自动分配一个端口转发到指定的端口。  
6). ENV:格式为 ENV <key> <value>。 指定一个环境变量，会被后续 RUN 指令使用，并在容器运行时保持。这就对应程序语言中的变量定义，可在需要的时候引用。例如：
<pre>
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
</pre>  
7). ADD:格式为 ADD src dest
该命令将复制指定的src到容器中的dest。 其中src可以是Dockerfile所在目录的一个相对路径；也可以是一个 URL；还可以是一个tar文件(自动解压为目录)  
8). COPY:格式为 COPY src dest
复制本地主机的src(为 Dockerfile所在目录的相对路径)到容器中的dest。当使用本地目录为源目录时，推荐使用 COPY  
COPY和ADD的不同就是：ADD多了自动解压和支持URL路径的功能。  
9). ENTRYPOINT：
两种格式：
 `ENTRYPOINT ["executable", "param1", "param2"]`
 `ENTRYPOINT command param1 param2(shell中执行)。`
配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖。
每个Dockerfile 中只能有一个 ENTRYPOINT，当指定多个时，只有最后一个起效。
CMD和ENTRYPOINT比较：两个命令都是只能使用一次，并且都是在执行docker run指令时运行，如果有多个，只执行最后一条。
两者的不同在于参数的传递方式不同  
10). VOLUME:格式为 VOLUME ["/data"]。创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。不过此属性在Dockerfile中指定并没有什么意义，因为没有办法指定本地主机的目录。如果需要指定挂载点可以在执行docker run命令时指定： 
 `docker run -it -v /home/fengzheng/ftp/:/data  859666d51c6d /bin/bash`  
11). USER:格式为 USER daemon。指定运行容器时的用户名或 UID，后续的 RUN 也会使用指定用户。
当服务不需要管理员权限时，可以通过该命令指定运行用户  
12). WORKDIR:格式为 WORKDIR /path/to/workdir。为后续的 RUN、CMD、ENTRYPOINT 指令配置工作目录。可以使用多个WORKDIR指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。例如:
<pre>    
WORKDIR /a
WORKDIR b
WORKDIR c
</pre>
RUN pwd
则最终路径为/a/b/c  
13). ONBUILD:格式为 ONBUILD [INSTRUCTION]。
配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。
例如，Dockerfile使用如下的内容创建了镜像 image-A
<pre> 
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
</pre>
如果基于image-A创建新的镜像时，新的Dockerfile中使用FROM image-A指定基础镜像时，会自动执行ONBUILD指令内容，等价于在后面添加了两条指令
<pre> 
FROM image-A
#Automatically run the following
ADD . /app/src
RUN /usr/local/bin/python-build --dir /app/src
</pre>
使用 ONBUILD 指令的镜像，推荐在标签中注明，例如 ruby:1.9-onbuild  
## 创建镜像 ##  
编写完Dockerfile后，通过docker build来创建镜像，基本格式为docker build [OPTIONS] 路径，它将读取指定路径(包括子路径)下的Dockerfile， 并将该路径下的所有内容发给Docker服务器， 由服务器来创建镜像， 建议放Dockerfile的目录为空目录，但也可以通过.dockerignore文件来让Docker忽略路径下的目录和文件  
要指定生成的镜像和标签信息，可以通过-t选项：  
 `docker build -t build_repo/first_image /tmp/docker_builder`
指定Dockerfile所在路径为/tmp/docker_builder, 并且希望生成的镜像标签为build_repo/first_image
## 使用Dockerfile创建ssh镜像 ##
### 编写Dockerfile ###
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
### 创建镜像 ###
在sshd_ubuntu目录下，创建sshd镜像，镜像名为sshd:dockerfile
<pre>
cd sshd_ubuntu
docker build -t sshd:dockerfile .
</pre>
命令执行完成，当看到有Successful build XXX字样，代表镜像创建成功
### 测试镜像 ###
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




