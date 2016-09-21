# 仓库 #
仓库是集中存放镜像的地方。  
注册服务器是存放仓库的具体服务器，每个服务器上有多个仓库，而仓库下面有多个镜像。例如对于仓库地址dl.dockerpool.com/ubuntu来说， dl.dockerpool.com是注册服务器地址，ubunbu是仓库名。  
## Docker Hub ##
官方公共仓库， 镜像根据是否为官方提供，将镜像分为两类：  
一类是类拟于centos等基础镜像，由Docker公司创建和维护，这样的镜像往往使用单词作为名字。  
一类是DockerHub用户创建并维护的，例如tianon/centos，带有用户名为前缀，表示是某用户的仓库。
## Docker Pool ##
国内专业Docker技术社区，也提供官方镜像的下载服务
### 查看镜像 ###
www.dockerpool.com/downloads  
### 下载镜像 ###
 `docker pull dl.dockerpool.com:5000/ubuntu:12.04`  
查看镜像：  
docker images  
修改标签与官方保持一致:  
 `docker tag dl.dockerpool.com:5000/ubuntu:12.04 ubuntu:12.04`  
## 创建和使用私有仓库 ##
### 使用registry镜像创建私有仓库 ###
安装完docker后，通过官方提供的registry镜像来搭建一套本地的私有仓库：  
 `docker run -d -p 5000:5000 registry`  
默认情况下，仓库会创建在容器的/tmp/registry目录下，可以通过-v参数指定把镜像文件放在本地的指定路径上，如将上传的镜像放在本地的/opt/data/registry下，在192.168.1.192上执行：  
 `docker run -d -p 5000:5000 -v /opt/data/registry:/tmp/registry registry` 此时本地启动一个私有仓库服务，端口为5000
### 管理私有仓库镜像 ###
在虚拟机中使用docker tag将镜像标记为192.168.1.192:5000/test  
 `docker tag centos:latest 192.168.1.192:5000/test`  
使用docker push上传标识的镜像  
 `docker push 192.168.1.192:5000/test`  
**在docker1.3以后的版本，使用https上传，所在在执行docker push操作的机器上，启动docker的方式必须为`docker -d --insecure-registry 192.168.1.192:5000 &`**  
用curl查看仓库192.168.1.192:5000中的镜像   
 `curl http://192.168.1.192:5000/v1/search`  
至此，可以到任何一台可以访问192.168.1.192的机器上去下载这个镜像了  
下载后，还可以添加更通用的标签，`docker tag 192.168.1.192:5000/test centos:latest`  


