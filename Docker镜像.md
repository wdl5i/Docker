# Docker镜像 #
## 获取镜像 ##
读者可以使用docker pull从网络上下载镜像， 命令格式为docker pull NAME[:TAG]。 如果不显示指定TAG, 默认会选择lastest标签， 即下载仓库中最新版本的镜像。  
docker pull ubuntu 相当于执行docker pull registry.hub.docker.com/ubuntu:lastest命令，也可以从国内下载镜像，如docker pull dl.dockerpool.com:5000/ubuntu:14.04  
<pre>
[root@192 ~]# docker pull ubuntu
latest: Pulling from ubuntu
f1b49dd0c243: Pull complete 
008ecf8686ec: Pull complete 
fd74137ff5ae: Pull complete 
35371c8124e2: Pull complete 
99dc4d8f603d: Pull complete 
37b164bb431e: Pull complete 
Digest: sha256:0cdb7cdc41a288622718832742b4fa935a528cdcaa3f71d7574c2f89ff0d516e
Status: Downloaded newer image for ubuntu:latest 
</pre>
下载过程可以看出，镜像文件由若干层组成，首列的008ecf8686ec字样代表了各层ID，层是AUFS(一种联合文件系统)的重要概念，是实现增量保存和更新的基础。  
下载镜像到本地后，就可以使用镜像了，例如用该镜像创建一个容器， 在其中运行bash应用：
`docker run -it ubuntu /bin/bash`
## 查看镜像信息 ##
使用docker images命令可以列出本地主机上已经的镜像 
<pre>
Status: Downloaded newer image for ubuntu:latest
[root@192 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              latest              37b164bb431e        3 weeks ago         126.6 MB
仓库                标签信息               镜像ID唯一           创建时间             大小
</pre>
可以使用docker tag为本地的镜像添加新的标签，如`docker tag dl.dockerpool.com:5000/ubuntu:lastest ubuntu:lastest`为tag dl.dockerpool.com:5000/ubuntu:lastest增加新的标签，相当于创建一个快捷方式。  
使用docker inspect命令可以获取镜像的详细信息：`docker inspect 37b164bb431e`
<pre> 
[root@192 ~]# docker inspect 37b164bb431e
[
{
    "Id": "37b164bb431e0119d4b891a3487e4b80977c56d838ff367966d31fc392c3a76d",
    "Parent": "99dc4d8f603d00196cee65a12a67dab905e781bdadb841d5a13fc08e44039041",
    "Comment": "",
    "Created": "2016-08-26T18:50:27.629952966Z",
    "Container": "1ff47ae2a4ea4e3ca166eb799d9e4a59c953787577dc2e892b19b544ad290727",
    "ContainerConfig": {
        "Hostname": "3934ed318998",
        "Domainname": "",
        "User": "",
        "AttachStdin": false,
        "AttachStdout": false,
        "AttachStderr": false,
        "PortSpecs": null,
        "ExposedPorts": null,
        "Tty": false,
        "OpenStdin": false,
        "StdinOnce": false,
        "Env": [
            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
        ],
        "Cmd": [
            "/bin/sh",
            "-c",
            "#(nop) CMD [\"/bin/bash\"]"
        ],
        "Image": "sha256:16f57b8272173310ae88c29b8dcc337624e1d00054d0279309eccb6d40793766",
        "Volumes": null,
        "VolumeDriver": "",
        "WorkingDir": "",
        "Entrypoint": null,
        "NetworkDisabled": false,
        "MacAddress": "",
        "OnBuild": null,
        "Labels": {}
    },
    "DockerVersion": "1.10.3",
    "Author": "",
    "Config": {
        "Hostname": "3934ed318998",
        "Domainname": "",
        "User": "",
        "AttachStdin": false,
        "AttachStdout": false,
        "AttachStderr": false,
        "PortSpecs": null,
        "ExposedPorts": null,
        "Tty": false,
        "OpenStdin": false,
        "StdinOnce": false,
        "Env": [
            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
        ],
        "Cmd": [
            "/bin/bash"
        ],
        "Image": "sha256:16f57b8272173310ae88c29b8dcc337624e1d00054d0279309eccb6d40793766",
        "Volumes": null,
        "VolumeDriver": "",
        "WorkingDir": "",
        "Entrypoint": null,
        "NetworkDisabled": false,
        "MacAddress": "",
        "OnBuild": null,
        "Labels": {}
    },
    "Architecture": "amd64",
    "Os": "linux",
    "Size": 0,
    "VirtualSize": 126608949 
}
]
</pre>
如果只需要其中某项属性信息，加-f参数来指定，若获取Architecture属性：
<pre>
[root@192 ~]# docker inspect -f {{.Architecture}} 37b164bb431e	
amd64
</pre>
## 搜索镜像 ##
用法为docker search TEAM，如搜索带mysql关键字的镜像:`docker search mysql`
## 删除镜像 ##
用法为docker rmi IMAGE[IMAGE...], 其中IMAGE可以用镜像ID和标签，如同一镜像下有多个标签，只是标签指定标签，并不会在删除镜像，如果一个镜像只有一个标签，删除该标签时，在物理上就删除了该镜像，即会删除该镜像的所有AUFS层， 当该镜像创建的容器存在时，是无法删除镜像的。正确做法是先删除该镜像的所有容器，再来删除镜像。  
使用docker ps -a 查看本机存在的所有容器：
<pre>
[root@192 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
4eb742b9b61f        ubuntu:latest       "/bin/bash"         17 minutes ago      Up 17 minutes                           admiring_pasteur   
</pre>
<code>
docker rm 4eb7   删除容器  
docker rmi 37b164bb431e   删除image
</code>
## 创建镜像 ##
创建镜像的三种方式：  
1. **基于已有镜像创建**  
主要使用docker commit命令， 命令格式为docker commit [OPTIONS] CONTAINER [REPOSTITORY[:TAG]], 主要选项包括：  
-a, --author="" 作者信息  
-m, --message="" 提交信息  
-p, --pause=true 提交时暂停容器运行  
步骤为：  
1). 创建新镜像，首先启动一下镜像，并在容器中进行修改
<pre>
[root@192 ~]# docker run -d -it ubuntu /bin/bash
3c6f5d0c90c79e43428166a82407bc09da0b692a0c11e40e829fdfdd3a4f6a33
[root@192 ~]# docker exec -it 3c /bin/bash
root@3c6f5d0c90c7:/# touch test.txt
root@3c6f5d0c90c7:/# exit 
</pre>
2). 使用docker commit来提交一个新的镜像,命令格式为：  
`docker commit [OPTIONS] containerId new_resp_name` 例如：  
`docker commit -m "add a new file test.txt" -a "wdl" 3c6f5d0c9 commit_test`  
执行docker images可以看到新提交的镜像已经存在于本地
<pre>
[root@192 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
commit_test         latest              657f5f8f5ab6        15 seconds ago      126.6 MB
ubuntu              latest              37b164bb431e        3 weeks ago         126.6 MB
</pre> 
2. **基于本地模板创建**  
推荐使用OpenVz的模板来导入，下载好模板好，执行命令：
`cat ubuntu-14.04-x86_64-minimal.tar.gz | docker import - ubuntu:14.04`
3. **基于Dockfile创建**  
使用docker commit创建镜像虽然简单，但是不方便团队合作，我们可以用docker build来创建一个镜像。
1). 新建目录和Dockerfile  
<pre>
[root@192 ~]# mkdir wdl
[root@192 ~]# cd wdl/
[root@192 wdl]# touch Dockerfile
</pre>
2). 编辑Dockerfile
<pre>
#This is a comment
FROM ubuntu:latest
MAINTAINER Docker wdl <wdl5i@163.com>
RUN apt-get -qq update 
RUN apt-get -qqy install ruby ruby-dev
RUN gem install wdl_docker
</pre>
使用#来注释  
FROM用于告诉Docker使用哪个镜像作为基础  
接着是维护者的信息  
RUN开头的指令会在创建中运行，比如安装一个包， 在这里使用apt-get安装一些软件
3). 使用docker build来生成镜像  
`docker build -t="用户TAG信息" Dockerfile路径` 如：  
`docker build -t="wdl/dockertest:v1" .`  
其中，-t用来标识tag, 指定新的镜像用户信息， .是指Dockerfile所在路径
## 上传镜像 ##
用户通过docker push命令，可以将自已创建的镜像上传到仓库中来共享， 例如，用户在docker hub上完成注册后，可以推送自已的镜像到仓库中。 命令格式为
 `docker push NAME[:TAG]`   
 `docker push wdl/dockertest:v1` 
## 存出和载入镜像 ##
存出镜像：  
如果要存出镜像到本地文件，使用docker save命令，如存出本地的ubuntu14.04到ubuntu_14.04.tar:  
 `docker save -o ubuntu_14.04.tar ubuntu:14.04`  
载入镜像： 
可以使用docker load从存出的本地文件中再导入到本地镜像库，如:  
 `docker load < ubuntu_14.04.tar`  

**删除所有tag为none的镜像：**  
<pre>
docker ps -a | grep "Exited" | awk '{print $1 }'|xargs docker stop
docker ps -a | grep "Exited" | awk '{print $1 }'|xargs docker rm
docker images|grep none|awk '{print $3 }'|xargs docker rmi
</pre>