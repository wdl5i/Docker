# Docker的核心概念和安装 #
## Docker核心概念 ##
**Docker镜像**  
Docker 是 PaaS 提供商 dotCloud 开源的一个基于 LXC 的高级容器引擎，源代码托管在 Github 上, 基于go语言并遵从Apache2.0协议开源。  
任何一个在互联网上提供其服务的公司都可以叫做云计算公司。其实云计算分几层的，分别是Infrastructure（基础设施）-as-a-Service，Platform（平台）-as-a-Service，Software（软件）-as-a-Service。基础设施在最下端，平台在中间，软件在顶端。别的一些“软”的层可以在这些层上面添加。  
![](http://i.imgur.com/GvdNBjJ.png) 

Docker镜像类似于虚拟机镜像，可以理解为面向Docker引擎的只读模板，包含了文件系统。镜像是创建Docker容器的基础，通过版本管理和增量的文件系统，Docker提供了一套十分简单的机制来创建和更新现有镜像。  
**Docker容器** 
类似于轻量级的安全沙箱，通过容器来运行和隔离应用。  
容器是从镜像创建的应用运行实例，可以将其启动，开始，停止，删除，而这些容器都是相互隔离的。  
镜像本身是只读的，容器从镜像启动的时候，Docker会在镜像最上层创建一个可写层，镜像本身将保持不变。  
**Docker仓库**  
类似于代码仓库，是Docker集中存放镜像文件的场所。**注册服务器**是存放仓库的地方，其上往往存放着多个仓库。每个仓库集中存放着某一类镜像， 往往包含多个镜像文件，通过不同的tag来区别。 如ubuntu操作系统仓库往往包括12.04, 14.04等不同版本的镜像  
![](http://i.imgur.com/1CgkDLE.png)
根据镜像公开与否， Docker仓库分为公开仓库和私有仓库两种形式。 当用户创建好自已的镜像后可以通过push命令将它上传到公有或私有仓库，这样用户在其他机器上使用该镜像时，只需要将其从仓库上pull下来就可以了。
## Docker安装 ##
### Ubuntu14.04及以上版本 ###
### Windows ###
### OSX ###
### Centos ###
**centos6.5**
<pre>  
yum install -y http://mirrors.yun-idc.com/epel/6/i386/epel-release-6-8.noarch.rpm
yum install -y docker-io 
</pre> 
**centos7及以上**  
`yum install -y docker`  


 