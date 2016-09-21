# 容器 #
容器就是镜像的一个运行实例，但它带有额外的可写文件层。  
## 创建容器 ##
创建：  `docker create -it ubuntu:latest` 新创建的容器处于停止状态  
启动：  `docker start 容器ID` 启动停止状态下的容器  
创建并启动：  `docker run`  
下面的命令输出一个hello, 之后容器自动停止：  
 `docker run ubuntu /bin/echo/ 'hello'`  
hello  
使用docker run来创建并启动容器时，会执行以后标准操作:  
1). 检查本地是否存在指定的镜像，不存在就从公网下载  
2). 利用镜像创建并启动容器  
3). 分配一个文件系统，并在镜像的外层挂一层可读写层  
4). 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中  
5). 从地址池配置一个IP给容器  
6). 执行用户指定的程序  
7). 执行程序完毕并终止容器  
下面的命令启动一个bash终端，允许用户进行交互：  
 `docker run -it ubuntu /bin/bash`  
其中， -t让docker分配一个伪终端并绑定到容器的标准输入上， -i则让容器的标准输入保持打开。  
用户可以使用ctrl + D或输入exit来退出容器， 退出后容器停止，因为当运行的应用退出后，容器没有继续运行的必要。用户可以输入-d参数使容器在后台以守护状态进行。  如下：
 `docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"`  
要获取容器的输出信息，使用`docker logs 容器ID`, 如：
<pre>
docker logs ce4
hello world
hello world
...
</pre>
## 终止容器 ##
docker stop命令格式为`docker stop [-t--time[=10]]`  
此外，当Docker容器指定的应用终结果，容器也自动终止 
docker restart 命令会先终止一个运行状态下的容器，然后再启动它. 
## 进入容器 ##
可以使用包括docker attach命令， docker exec命令, 及nsenter工具等
**attach**: 当多个窗口同时attach到同一个容器时，所有窗口会同步显示 ，阻塞时会一起阻塞。  
**exec**: 如`docker exec -ti 342344 /bin/bash`  
**nsenter工具**  
## 删除容器 ##
使用docker rm删除处于停止状态下的容器，格式为`docker rm [OPTIONS] CONTAINER [CONTAINER...]`, 选项包含：  
-f: --force=false, 强止终止并删除一个运行中的容器， Docker会发送SIGKILL信号给容器， 终止其中的应用。    
-l: --link=false, 删除容器的连接，保留容器  
-v: --volumes=false, 删除容器挂载的数据卷   
## 导入导出容器 ## 
**导出容器**是指导出一个已经创建的容器到本地文件，不管此时容器是否正在运行，命令为docker export，格式为`docker export CONTAINER > 文件地址`， 如
 `docker export ce5 > test_for_run.tar`  
可将这些文件传输到其他机器上，在其他机器通过导入命令实现容器的迁移。  
**导入容器**，可以使用命令docker import 将文件导入为镜像。  
 `cat test_for_run.tar | docker import - test/ubuntu:v1`  
docker import和docker load都可以实现导入镜像文件，区别在于：   
docker import导入的是快照，是没有历史记录的，体积小  
docker load导入的是镜像，是有历史记录的，体积大 

