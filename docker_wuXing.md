本文作者：**wuXing**
QQ:** 1226032602**
E-mail:** 1226032602@qq.com**

# **docker**

1. **什么是LXC**

LXC为Linux Container的简写。Linux Container容器是一种内核虚拟化技术，可以提供轻量级的虚拟化，以便隔离进程和资源，而且不需要提供指令解释机制以及全虚拟化的其他复杂性。相当于C++中的NameSpace。容器有效地将由单个操作系统管理的资源划分到孤立的组中，以更好地在孤立的组之间平衡有冲突的资源使用需求。与传统虚拟化技术相比，它的优势在于：
与宿主机使用同一个内核，性能损耗小；
不需要指令级模拟；
不需要即时(Just-in-time)编译；
容器可以在CPU核心的本地运行指令，不需要任何专门的解释机制；
避免了准虚拟化和系统调用替换中的复杂性；
轻量级隔离，在隔离的同时还提供共享机制，以实现容器与宿主机的资源共享。

总结：Linux Container是一种轻量级的虚拟化的手段。


 
2. **什么是Docker**

Docker是Docker.inc公司开源的一个基于LXC技术之上构建的Container容器引擎，源代码托管在GitHub上，基于Go语言并遵从Apache2.0协议开源（可以商业）。
Docker项目的目标是实现轻量级的操作系统虚拟化解决方案。
Docker是通过内核虚拟化技术（namespaces及cgroups等）来提供容器的资源隔离与安全保障等。由于Docker通过操作系统层的虚拟化实现隔离，所以Docker容器在运行时，不需要类似虚拟机VM额外的操作系统开销，提高资源利用率。
下面图比较了Docker和传统虚拟化方式的不同之处，可见容器是在操作系统层面上实现虚拟化，直接复制本地主机的操作系统，而传统方式则是在硬件层面实现。




![图片](https://images-cdn.shimo.im/M71OLoBbmqoaMvCE/image.png!thumbnail)




![图片](https://images-cdn.shimo.im/tuvZRmXxxlQ0wiXC/image.png!thumbnail)







3. ** Docker的工作模式**

学习Docker的源码并不是一个枯燥的过程，反而可以从中理解Docker架构的设计原理。
Docker对使用者来讲是一个C/S模式的架构，而Docker的后端是一个非常松耦合的架构，模块各司其职，并有机组合，支撑Docker的运行。
用户是使用Docker Client与Docker Daemon建立通信，并发送请求给后者。
而Docker Daemon作为Docker架构中的主体部分，首先提供Server的功能使其可以接受Docker Client的请求；而后Engine执行Docker内部的一系列工作，每一项工作都是以一个Job的形式的存在。
Job的运行过程中，当需要容器镜像时，则从Docker Registry中下载镜像，并通过镜像管理驱动graphdriver将下载镜像以Graph的形式存储；当需要为Docker创建网络环境时，通过网络管理驱动networkdriver创建并配置Docker容器网络环境；当需要限制Docker容器运行资源或执行用户指令等操作时，则通过execdriver来完成。而libcontainer是一项独立的容器管理包，networkdriver以及execdriver都是通过libcontainer来实现具体对容器进行的操作。当执行完运行容器的命令后，一个实际的Docker容器就处于运行状态，该容器拥有独立的文件系统，独立并且安全的运行环境等。


## **CLI交互模型**
![图片](https://images-cdn.shimo.im/QaKjQvxHc0ca4gKM/image.image/png!thumbnail)



4. ** ****Docker八中应用场景**

1、简化配置,统一配置,通过镜像快速启动(Simplifying)
2、代码流水线管理,开发环境->测试环境->预生产环境->灰度发布->正式发布，docker在这里实现了快速迁移(Code Oioeline Management)
3、开发效率,对开发人员,有了镜像,直接启动容器即可(Developer Productivity)
4、应用隔离,相对于虚拟机的完全隔离会占用资源,docker会比较节约资源(App lsolation)
5、服务器整合,一台服务器跑多个docker容器,提高服务器的利用率(Server Consolidation)
6、调试能力,debug调试(Debugging Capabilties)
7、多租户,一个租户多个用户,类似于阿里公有云的一个project下多个用户(Multi-tenancy)
8、快速部署,不需要启动操作系统,实现秒级部署(Rapid Deplovment)




5. ** Docker八中开发模式**

1.共享基础容器
2.共享卷开发容器
3.开发工具容器
4.不同环境下测试容器
5.构建容器
6.安装容器
7.盒子中默认服务容器
8.基础设施/粘合剂容器



6. ** Docker九个基本事实**

1.容器不同于虚拟机
2.容器不如虚拟机来得成熟
3.容器可以在几分之一秒内启动
4.容器已在大规模环境证明了自身的价值
5.IT人员称容器为轻量级
6.容器引发了安全问题
7.Docker已成为容器的代名词，但它不是唯一的提供者
8.容器能节省IT人力，加快更新
9.容器仍面临一些没有解决的问题






7. ** 使用Docker理由**

作为一种新兴的虚拟化方式，Docker跟传统的虚拟化方式具有众多的优势。
首先，Docker容器的启动可以在秒级实现，这相比传统的虚拟机方式要快得多。其次，Docker对系统资源的利用率很低，一台主机上可以同时运行数千个Docker容器。
至于为什么要使用Docker：
**技术储备**
相对大公司这个非常重要,如果你们都在用,他们不用就落后了,等到完全成熟以后就跟不上了。
**无技术栈和技术债**
没有任何Openstack或者saltstack,服务down了就down了,所有的服务都是松耦合。
**3、跟上潮流(提升自我,装逼)**
面试的时候大家都会Docker,你不会是不是落后了。
**4、符合当前业务**
虽然Docker很优秀,但是，大多数处在第二种状态,很少有符合自己的业务


8. ** Docker改变了什么**

面向产品：产品交付
面向开发：简化环境配置
面向测试：多版本测试
面向运维：环境一致性
面向架构：自动化扩容(微服务)






9. ** Docker更快速的交付和部署**

对于开发和人员来说，最希望的就是一次创建和配置，可以在任意地方正常运行。
开发者可以使用一个标准的镜像来构建一套开发容器，开发完成之后，运维人员可以直接使用这个容器来部署代码。Docker可以快速创建容器，快速迭代应用程序，并让整个过程全称可见，使团队中的其他成员更容易理解应用程序是如何创建和工作。Docker容器很轻很快！容器的启动时间是秒级的，大量第节约开发、测试、部署的时间。

10. ** Docker更高效的虚拟化**

Docker容器的运行不需要额外的Hypervisor支持，它是内核级的虚拟化，因此可以实现更高的性能和效率。

11. ** Docker更轻松的迁移和扩展**

Docker容器几乎可以在任意的平台上运行，包括物理机、虚拟机、公有云、私有云、个人电脑、服务器等。这种兼容性可以让用户把一个应用程序从一个平台直接迁移到另外一个。

12. ** Docker更简单的管理**

使用Docker，只需要小小的修改，就可以替代往大量的更新工作。所有的修改都以增量的方式被分发和更新，从而实现自动化并且高效的管理。







13. ** ****Docker与虚拟化**

|    |    |    |    | 
|:----|:----:|:----|:----:|:----|:----:|:----|:----:|
| 类别   | Docker   | OpenStack   | 结论   | 
| 部署难度   | 非常简单   | 组件多，部署复杂   | 因为平台是对已有的线上生产环境进行改造，必须选择侵入性较小的容器化技术   | 
| 启动速度   | 秒级   | 分钟级   | 面对流量峰值,速度就是一切   | 
| 执行性能   | 和物理系统几乎一致   | VM会占用一些资源   | 微博核心业务对服务SLA要求非常苛刻   | 
| 镜像体积   | 镜像是MB级别   | 虚拟机镜像是GB级别   | 当集群大规模部署时，体积小就代表更大的并发调度量   | 
| 管理效率   | 管理简单   | 组件相互依赖，管理复杂   | 生产系统集群可控性是核心竞争力   | 
| 隔离性   | 隔离性高   | 彻底隔离   |    | 
| 可管理性能   | 单进程、不建议启动SSH   | 完整的系统管理   |    | 
| 网络连接   | 比较弱   | 借助Neutron可以灵活组建各类网络架构   |    | 






![图片](https://images-cdn.shimo.im/nuPWujMlgU4jOSQi/image.image/png!thumbnail)




14. ** ****Docker三大核心概念**
1. ** Docker镜像(image)**

Docker镜像就是一个只读的模板。
例如：一个镜像可以包含一个完整的CentOS操作系统环境，里面仅安装了Apache或用户需要的其他应用程序。
镜像可以用来创建Docker容器。
Docker提供了一个很简单的机制来创建镜像或者更新现有的镜像，用户甚至可以直接从其他人那里下载一个已经做好的镜像来直接使用。

2. ** Docker容器(container)**

Docker利用容器来运行应用。
容器是从镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的，保证安全的平台。
可以把容器看做是一个简易版的Linux环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。
注意：镜像是只读的，容器在启动的时候创建一层可写层作为最上层。

3. ** Docker仓库(repository)**

仓库是集中存放镜像文件的场所。有时候把仓库和仓库注册服务器（Registry）混为一谈，并不严格区分。实际上，仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签(tag)。

仓库分为公开仓库(Public)和私有仓库(Private)两种形式。

最大的公开仓库是Docker Hub，存放了数量庞大的镜像供用户下载。国内的公开仓库包括Docker Pool等，可以提供大陆用户更稳定快读的访问
。
当用户创建了自己的镜像之后就可以使用push命令将它上传到公有或者私有仓库，这样下载在另外一台机器上使用这个镜像时候，只需需要从仓库上pull下来就可以了。

注意：Docker仓库的概念跟Git类似，注册服务器可以理解为GitHub这样的托管服务






15. ** docker安装**


[https://yq.aliyun.com/articles/110806](https://yq.aliyun.com/articles/110806)



```
# step 1: 安装必要的一些系统工具
yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
yum makecache fast
yum -y install docker-ce
# Step 4: 开启Docker服务
service docker start
```


```
# 注意：
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，你可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ce.repo
#   将 [docker-ce-test] 下方的 enabled=0 修改为 enabled=1
#
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2 : 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]
```


fedora安装

```
dnf remove docker docker-client docker-client-latest              docker-common docker-latest docker-latest-logrotate               docker-logrotate docker-selinux docker-engine-selinux                 docker-engine

dnf -y install dnf-plugins-core
dnf config-manager --add-repo     https://download.docker.com/linux/fedora/docker-ce.repo

dnf config-manager --set-enabled docker-ce-edge
dnf config-manager --set-enabled docker-ce-test
dnf config-manager --set-disabled docker-ce-edge
dnf install docker-ce
```



[https://cr.console.aliyun.com](https://cr.console.aliyun.com/cn-qingdao/repositories)



配置文件
cat /etc/docker/daemon.json
```
{
  "registry-mirrors": ["https://gqk8w9va.mirror.aliyuncs.com"],
  "graph":"/opt/mydocker"
}
```


启动
```
systemctl start docker
```




查看版本信息
```
[root@centos72 ~]# docker version
Client:
 Version:         1.12.6
 API version:     1.24
 Package version: docker-common-1.12.6-11.el7.centos.x86_64
 Go version:      go1.7.4
 Git commit:      96d83a5/1.12.6
 Built:           Tue Mar  7 09:23:34 2017
 OS/Arch:         linux/amd64

Server:
 Version:         1.12.6
 API version:     1.24
 Package version: docker-common-1.12.6-11.el7.centos.x86_64
 Go version:      go1.7.4
 Git commit:      96d83a5/1.12.6
 Built:           Tue Mar  7 09:23:34 2017
 OS/Arch:         linux/amd64
```



16. ** ****Docker基础命令**

4. ** 帮助**

docker --help

A self-sufficient runtime for containers.

Options:

–config=~/.docker Location of client config files #客户端配置文件的位置

-D, –debug=false Enable debug mode #启用Debug调试模式

-H, –host=[] Daemon socket(s) to connect to #守护进程的套接字（Socket）连接

-h, –help=false Print usage #打印使用

-l, –log-level=info Set the logging level #设置日志级别

–tls=false Use TLS; implied by –tlsverify #

–tlscacert=~/.docker/ca.pem Trust certs signed only by this CA #信任证书签名CA

–tlscert=~/.docker/cert.pem Path to TLS certificate file #TLS证书文件路径

–tlskey=~/.docker/key.pem Path to TLS key file #TLS密钥文件路径

–tlsverify=false Use TLS and verify the remote #使用TLS验证远程

-v, –version=false Print version information and quit #打印版本信息并退出

Commands:

attach Attach to a running container #当前shell下attach连接指定运行镜像

build Build an image from a Dockerfile #通过Dockerfile定制镜像

commit Create a new image from a container’s changes #提交当前容器为新的镜像

cp Copy files/folders from a container to a HOSTDIR or to STDOUT #从容器中拷贝指定文件或者目录到宿主机中

create Create a new container #创建一个新的容器，同run 但不启动容器

diff Inspect changes on a container’s filesystem #查看docker容器变化

events Get real time events from the server #从docker服务获取容器实时事件

exec Run a command in a running container #在已存在的容器上运行命令

export Export a container’s filesystem as a tar archive #导出容器的内容流作为一个tar归档文件(对应import)

history Show the history of an image #展示一个镜像形成历史

images List images #列出系统当前镜像

import Import the contents from a tarball to create a filesystem image #从tar包中的内容创建一个新的文件系统映像(对应export)

info Display system-wide information #显示系统相关信息

inspect Return low-level information on a container or image #查看容器详细信息

kill Kill a running container #kill指定docker容器

load Load an image from a tar archive or STDIN #从一个tar包中加载一个镜像(对应save)

login Register or log in to a Docker registry #注册或者登陆一个docker源服务器

logout Log out from a Docker registry #从当前Docker registry退出

logs Fetch the logs of a container #输出当前容器日志信息

pause Pause all processes within a container #暂停容器

port List port mappings or a specific mapping for the CONTAINER #查看映射端口对应的容器内部源端口

ps List containers #列出容器列表

pull Pull an image or a repository from a registry #从docker镜像源服务器拉取指定镜像或者库镜像

push Push an image or a repository to a registry #推送指定镜像或者库镜像至docker源服务器

rename Rename a container #重命名容器

restart Restart a running container #重启运行的容器

rm Remove one or more containers #移除一个或者多个容器

rmi Remove one or more images #移除一个或多个镜像(无容器使用该镜像才可以删除，否则需要删除相关容器才可以继续或者-f强制删除)

run Run a command in a new container #创建一个新的容器并运行一个命令

save Save an image(s) to a tar archive #保存一个镜像为一个tar包(对应load)

search Search the Docker Hub for images #在docker hub中搜索镜像

start Start one or more stopped containers #启动容器

stats Display a live stream of container(s) resource usage statistics #统计容器使用资源

stop Stop a running container #停止容器

tag Tag an image into a repository #给源中镜像打标签

top Display the running processes of a container #查看容器中运行的进程信息

unpause Unpause all processes within a container #取消暂停容器

version Show the Docker version information #查看容器版本号

wait Block until a container stops, then print its exit code #截取容器停止时的退出状态值

Run ‘docker COMMAND –help’ for more information on a command. 







17. ** Docker镜像管理**

镜像是分层的，有两种类型的镜像层,最上层是可写的，其它都是只读的

[https://cr.console.aliyun.com](https://cr.console.aliyun.com/cn-qingdao/repositories)


```
cat  /etc/docker/daemon.json
{
  "registry-mirrors": ["https://gqk8w9va.mirror.aliyuncs.com"],
  "graph":"/opt/mydocker"
}
```

```
mkdir -p /opt/mydocker
systemctl daemon-reload
systemctl restart docker.service
```


docker pull nginx




5. ** 搜索Docker镜像**

搜索所有centos的docker镜像
```
docker search centos
```





6. ** 获取Docker镜像**

docker pull 镜像名

### 获取centos镜像
```
docker pull centos
```

### 获取centos6镜像
```
docker pull centos:6
docker pull tutum/centos
docker pull centos:7.2.1511
docker pull nginx
docker pull mysql
docker pull mysql:5.7
```




7. ** #使用镜像创建一个容器**

[root@centos72 ~]# **docker run -it centos /bin/bash**
[root@72119e05edf9 /]#






8. ** 查看docker镜像**
```
[root@centos72 ~]# docker images 
REPOSITORY (来自那个仓库)  TAG (标签) IMAGE ID (唯一ID) CREATED (创建时间) SIZE (大小)
docker.io/centos    latest              a8493f5f50ff        8 days ago          192.5 MB


REPOSITORY:来自于哪个仓库，比如 centos
TAG:镜像的标记，一般修改版本号
IMAGE ID:镜像的id号
CREATED：创建镜像的时间
VIRTUAL SIZE：镜像的大小
```




9. ** 查看镜像详细信息**
```
docker inspect 镜像名
```


```
docker image history --no-trunc nginx
```

```
docker history centos:latest
```


10. ** #删除容器**

```
[root@centos72 ~]# docker rm 9439d686a2af
9439d686a2af
```


11. ** ****删除Docker镜像**
```
[root@centos72 ~]# docker images          
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos    latest              a8493f5f50ff        8 days ago          192.5 MB
[root@centos72 ~]# docker rmi a8493f5f50ff -f
Untagged: docker.io/centos:latest
Untagged: docker.io/centos@sha256:4eda692c08e0a065ae91d74e82fff4af3da307b4341ad61fa61771cc4659af60
Deleted: sha256:a8493f5f50ffda70c2eeb2d09090debf7d39c8ffcd63b43ff81b111ece6f28bf
[root@centos72 ~]# docker images             
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

docker rmi `docker image ls -q`
```



12. ** 启动**

```
[root@centos72 ~]# docker start f30608dcb22e
f30608dcb22e
```

13. ** 停止**
```
[root@centos72 ~]# docker stop f30608dcb22e 
f30608dcb22e
```

14. ** 导出Docker镜像**
```
docker image save alpine:latest > docker-alpine.tar.gz
```

15. ** 导入Docker镜像**
```
docker image load -i docker-alpine.tar.gz
```




18. ** Docker容器管理**
16. ** 启动Docker容器**

启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态(stopped)的容器重新启动。
因为Docker的容器实在太轻量级了，很多时候用户都是随时删除和新创建容器。

17. ** 新建容器并启动**

所需要的命令主要为  docker run
#输出一个hehe,之后终止容器
```
[root@centos72 ~]# docker run centos /bin/echo "hehe"
hehe
```


#启动一个bash终端,允许用户进行交互
```
[root@centos72 ~]# docker run --name mydocker -it centos /bin/bash
[root@d1a97410ef96 /]# ls
anaconda-post.log  dev  home  lib64       media  opt   root  sbin  sys  usr
bin                etc  lib   lost+found  mnt    proc  run   srv   tmp  var
[root@d1a97410ef96 /]# pwd
/
```

--name:给容器定义一个名称
-i:则让容器的标准输入保持打开（开启交互式shell）
-t:让Docker分配一个伪终端,并绑定到容器的标准输入上（为容器分配一个伪tty终端）
centos:指定镜像的名字
/bin/bash:执行一个命令


docker run -it centos:6 /bin/bash

退出docker镜像，当你退出时镜像也停止了


当利用docker run来创建容器时，Docker在后台运行的标准操作包括：
**检查本地是否存在指定的镜像，不存在就从公有仓库下载**
**利用镜像创建并启动一个容器**
**分配一个文件系统，并在只读的镜像层外面挂在一层可读写层**
**从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去**
**从地址池配置一个ip地址给容器**
**执行用户指定的应用程序**
**执行完毕后容器被终止**



```
docker run --rm -it alpine sh

docker run -d --privileged=true --name mysql56 tutum/centos:latest /usr/sbin/init

docker run -d -p 80:80 --privileged=true --name web01 tutum/centos /usr/sbin/init

docker run -d --privileged=true -m 512m -h web01 --name web01 wuxingge/centos:v1 /usr/sbin/init

docker run -dit --privileged=true --name mycentos6 centos:6 /bin/bash

docker exec -it web01 /bin/bash

docker run -d -P --name web --link db:db training/webapp
```


db 容器和 web 容器建立互联关系。
--link 参数的格式为 --link name:alias ，其中 name 是要链接的容器的
名称， alias 是这个连接的别名

-p 7777:1080 -p 80:80



自定义镜像启动容器
```
docker run -d -p 80:80 --privileged=true --name web wuxinge/httpd:v1 /usr/sbin/init
```


```
docker run -it --rm -p 3306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql:5.7
```

```
docker run -it --rm -p 3306:3306 -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql:5.7
```


18. ** 查看容器**
```
[root@centos72 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                          PORTS               NAMES
d1a97410ef96        centos              "/bin/bash"         5 minutes ago       Exited (0) About a minute ago                       mydocker
0cfafa73a706        centos              "/bin/echo hehe"    9 minutes ago       Exited (0) 9 minutes ago                            sleepy_saha
f30608dcb22e        centos              "/bin/bash"         34 minutes ago      Exited (137) 27 minutes ago                         reverent_mccarthy
606d6edb45e4        centos              "/bin/bash"         50 minutes ago      Exited (0) 49 minutes ago                           jovial_pare
72119e05edf9        centos              "/bin/bash"         53 minutes ago      Exited (0) 50 minutes ago                           naughty_jang

docker ps -a -q
```


查看一个容器详细信息
```
docker container inspect 容器名（或ID）
```
## 









19. ** 守护进程运行**

更多的时候，需要让Docker容器在后台以守护形式运行。此时可以通过添加-d参数来实现

```
docker run -d centos /bin/bash -c "while true; do echo hehe; sleep 1;done"
```

```
[root@centos72 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
5067c13c1f36        centos              "/bin/bash -c 'while "   About a minute ago   Up About a minute                       prickly_wright
```

20. ** 获取容器输出信息**
```
docker logs 5067c13c1f36
```


21. ** 停止容器**
```
docker stop 5067c13c1f36
```

22. ** 列出所有启动容器的ID**
```
[root@centos72 ~]# docker ps -a -q
be76eaacccf1
```

23. ** ****批量杀掉启动的容器**
```
docker kill $(docker ps -a -q)
```

24. ** 删除容器**
```
docker rm be76eaacccf1
```

```
docker rm `docker ps -a -q`
```


25. ** 进入容器**

使用-d参数时，容器启动后会进入后台。某些时候需要进入容器进行操作,有很多种方法，包括使用docker attach命令或nsenter工具等
### **attach进入容器**
```
[root@centos72 ~]# docker run --name mydocker -it centos /bin/bash
[root@a8ecf72b4d1e /]# exit
[root@centos72 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
a8ecf72b4d1e        centos              "/bin/bash"         8 seconds ago       Exited (0) 2 seconds ago                       mydocker
[root@centos72 ~]# docker start a8ecf72b4d1e
a8ecf72b4d1e
[root@centos72 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
a8ecf72b4d1e        centos              "/bin/bash"         52 seconds ago      Up 4 seconds                            mydocker
[root@centos72 ~]# docker attach a8ecf72b4d1e
[root@a8ecf72b4d1e /]#
```

### **nsenter进入容器**
nsenter可以访问另一个进程的名字空间。nsenter需要有root权限
#安装包中有需要用到的nsenter
yum install -y util-linux

nsenter --help
...
-t, --target <pid>     target process to get namespaces from
-m, --mount[=<file>]   enter mount namespace
-u, --uts[=<file>]     enter UTS namespace (hostname etc)
-i, --ipc[=<file>]     enter System V IPC namespace
-n, --net[=<file>]     enter network namespace
-p, --pid[=<file>]     enter pid namespace
-U, --user[=<file>]    enter user namespace
-S, --setuid <uid>     set uid in entered namespace
-G, --setgid <gid>     set gid in entered namespace
    --preserve-credentials do not touch uids or gids
-r, --root[=<dir>]     set the root directory
-w, --wd[=<dir>]       set the working directory
-F, --no-fork          do not fork before exec'ing <program>
-Z, --follow-context   set SELinux context according to --target PID

1. ** #找到容器的第一个进程PID**

[root@centos72 ~]# docker inspect --format "{{.State.Pid}}" a8ecf72b4d1e
22082

2. ** #通过这个PID连接到容器**

[root@centos72 ~]# nsenter --target 22082 --mount --uts --ipc --net --pid
[root@a8ecf72b4d1e ~]#

nsenter -t 9618 -m -u -i -p -n

3. ** ****#编写成脚本快速进入容器空间**

cat in.sh 
#!/bin/bash
PID=$(docker inspect --format "{{.State.Pid}}" $1)
nsenter --target $PID --mount --uts --ipc --net --pid

4. ** #执行脚本跟上容器ID快速进入**

[root@centos72 ~]# ./in.sh a8ecf72b4d1e
[root@a8ecf72b4d1e ~]#

### **exec进入容器**
```
docker exec -it web01 /bin/bash
```



## **退出容器**
```
[root@5a293d35b4be /]# read escape sequence          #####  ctrl+pq  退出docker，仍然运行
[root@db01 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS               NAMES
5a293d35b4be        centos              "bash"              13 minutes ago      Up 2 minutes                                      myserver
2f566cfcaa5e        alpine              "/bin/sh"           26 minutes ago      Exited (137) 16 minutes ago                       test1
[root@db01 ~]#
```



26. ** 导出容器快照**

导出容器快照到本地文件
```
docker export 7691a814370e > centos.tar
```

27. ** 导入容器快照**

从容器快照文件中再导入为镜像
```
cat centos.tar |docker import - test/centos:v1.0
```


28. ** 容器制作镜像**
```
[root@mycentos ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS               NAMES
0b9beb964770        tutum/centos        "/usr/sbin/init"    About an hour ago   Exited (137) 17 minutes ago                       web01

[root@mycentos ~]# docker commit -m "centos http server" web01 wuxinge/httpd:v1
sha256:37d80ce7b6d91c05f50e8808ed57a9c4b28c2878c0246f953281874a7184b2ef
[root@mycentos ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
wuxinge/httpd       v1                  37d80ce7b6d9        17 seconds ago      450MB
tutum/centos        latest              99a633ad346f        20 months ago       297MB
```





19. ** docker网络**

man docker network


容器上网：
1、宿主机开启内核转发
2、iptables配置nat转换



29. ** 查看网络**
```
docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
8b7070748544        bridge              bridge              local
60d04beab36b        host                host                local
f04abfea71b5        none                null                local
```


host网络：容器使用宿主机的网卡
```
docker run -it --network=host busybox
```


30. ** 修改默认网络**
5. ** 删除默认的docker0网络**
```
ip link set dev docker0 down
brctl delbr docker0
iptables -t nat -F POSTROUTING
```

6. ** 添加新的默认网络**
```
brctl addbr bridge0
ip addr add 192.168.5.1/24 dev bridge0
ip link set dev bridge0 up
```

7. ** 修改配置文件**

cat /etc/docker/daemon.json 
```
{
  "registry-mirrors": ["https://gqk8w9va.mirror.aliyuncs.com"],
  "graph":"/opt/mydocker",
  "insecure-registries":["10.0.0.11:5000"],
  "bridge": "bridge0"
}
```

重启docker服务



8. ** 运行容器测试**
```
docker run -d --name test centos6-ssh-http
```

```
docker inspect test |grep -i ipaddr
            "SecondaryIPAddresses": null,
            "IPAddress": "192.168.5.2",
                    "IPAddress": "192.168.5.2"
```


31. ** 自定义网络**
9. ** 创建网络**
```
docker network create --driver bridge --subnet 192.168.1.0/24 --gateway 192.168.1.1 my_net

docker network create --driver bridge --subnet 10.0.0.0/24 --gateway 10.0.0.254 my_net
```


10. ** 运行容器测试**
```
docker run -d --network=my_net --ip 192.168.1.3 --name web centos6-ssh-http

docker run -d --network=my_net --ip 10.0.0.200 -h oldboy43 --name oldboy43 centos6:latest

docker run -d --privileged --network my_net --ip 10.0.0.51 -h db01 --name db01 centos7:latest /usr/sbin/init

docker inspect web |grep -i ipaddr
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "192.168.1.3",
```

32. ** 删除网络**
```
docker network rm my_net
```

33. ** 容器间通信**
```
docker run -it --network=tax_02 busybox
docker run -it --network=bridge busybox
```


docker network connect [OPTIONS] NETWORK CONTAINER
```
docker network connect tax_02 6254db90643c
docker network connect bridge 8657fbf3ff3e
```





34. ** dns网络**

只支持自定义网络

```
docker run -it --network=tax_02 --name=ops01 centos
ping ops02
docker run -it --network=tax_02 --name=ops02 centos
ping ops01
```

20. ** Docker Machine**

真正的环境中会有多个 host，容器在这些 host 中启动、运行、停止和销毁，相关容器会通过网络相互通信，无论它们是否位于相同的 host

192.168.56.101   安装 Docker Machine，然后通过 docker-machine 命令在其他两个 host 上部署 docker

实验环境：
192.168.56.101   Docker Machine
192.168.56.104
192.168.56.105

35. ** 安装 Docker Machine**

官方安装文档
https://docs.docker.com/machine/install-machine/

curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine &&
chmod +x /tmp/docker-machine &&
cp /tmp/docker-machine /usr/local/bin/docker-machine


为了得到更好的体验，我们可以安装 bash completion script，这样在 bash 能够通过 tab 键补全 docker-mahine 的子命令和参数。安装方法是从https://github.com/docker/machine/tree/master/contrib/completion/bash下载 completion script
![图片](https://images-cdn.shimo.im/M2QmEgc5dZ4HqeWo/image.image/png!thumbnail)


将其放置到 /etc/bash_completion.d 目录下
执行如下：
scripts=( docker-machine-prompt.bash docker-machine-wrapper.bash docker-machine.bash ); for i in "${scripts[@]}"; do sudo wget https://raw.githubusercontent.com/docker/machine/v0.13.0/contrib/completion/bash/${i} -P /etc/bash_completion.d; done

要启用docker-machineshell提示符，请添加$(__docker_machine_ps1)到您的PS1设置中~/.bashrc
PS1='[\u@\h \W$(__docker_machine_ps1)]\$ '
其作用是设置 docker-machine 的命令行提示符，不过要等到部署完其他两个 host 才能看出效果

36. ** 创建 Machine **

对于 Docker Machine 来说，术语 Machine 就是运行 docker daemon 的主机。“创建 Machine” 指的就是在 host 上安装和部署 docker

11. ** 查看当前machine**

[root@db01 ~]# docker-machine ls
NAME   ACTIVE   DRIVER   STATE   URL   SWARM   DOCKER   ERRORS


12. ** 创建 machine 要求能够无密码登录远程主机**

ssh-keygen -t rsa
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.56.104
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.56.105


13. ** 执行 docker-machine create 命令创建 host1、host2**

提示：host1和host2准备好docker yum源
docker-machine create --driver generic --generic-ip-address=192.168.56.104 host1
docker-machine create --driver generic --generic-ip-address=192.168.56.105 host2




14. ** docker daemon 的具体配置**

cat /etc/systemd/system/docker.service.d/10-machine.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2376 -H unix:///var/run/docker.sock --storage-driver devicemapper --tlsverify --tlscacert /etc/docker/ca.pem --tlscert /etc/docker/server.pem --tlskey /etc/docker/server-key.pem --label provider=generic 
Environment=


/usr/lib/systemd/system/docker.service
...
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0


docker -H 10.0.0.11 images
docker -H 10.0.0.11 info


37. ** 管理 Machine **
15. ** 查看host需要的环境变量**

[root@db01 ~]# docker-machine env host1
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.56.104:2376"
export DOCKER_CERT_PATH="/root/.docker/machine/machines/host1"
export DOCKER_MACHINE_NAME="host1"
# Run this command to configure your shell: 
# eval $(docker-machine env host1)

[root@db01 ~]# eval $(docker-machine env host1)
[root@db01 ~ [host1]]#


16. ** docker-machine upgrade 更新 machine 的 docker 到最新版本**

可以批量执行
docker-machine upgrade host1 host2





17. ** 查看 machine 的 docker daemon 配置**

docker-machine config host1
--tlsverify
--tlscacert="/root/.docker/machine/machines/host1/ca.pem"
--tlscert="/root/.docker/machine/machines/host1/cert.pem"
--tlskey="/root/.docker/machine/machines/host1/key.pem"
-H=tcp://192.168.56.104:2376


stop/start/restart 是对 machine 的操作系统操作，而 不是 stop/start/restart  docker daemon。

docker-machine scp 可以在不同 machine 之间拷贝文件，比如：
docker-machine scp host1:/tmp/a host2:/tmp/b





21. ** 跨主机网络**

跨主机网络方案包括
docker 原生的 overlay 和 macvlan。
第三方方案：常用的包括 flannel、weave 和 calico。

docker 网络是一个非常活跃的技术领域，不断有新的方案开发出来，那么要问个非常重要的问题了：
如此众多的方案是如何与 docker 集成在一起的？
答案是：libnetwork 以及 CNM。

libnetwork & CNM
libnetwork 是 docker 容器网络库，最核心的内容是其定义的 Container Network Model (CNM)，这个模型对容器网络进行了抽象，由以下三类组件组成：

Sandbox
Sandbox 是容器的网络栈，包含容器的 interface、路由表和 DNS 设置。 Linux Network Namespace 是 Sandbox 的标准实现。Sandbox 可以包含来自不同 Network 的 Endpoint。

Endpoint
Endpoint 的作用是将 Sandbox 接入 Network。Endpoint 的典型实现是 veth pair，后面我们会举例。一个 Endpoint 只能属于一个网络，也只能属于一个 Sandbox。

Network
Network 包含一组 Endpoint，同一 Network 的 Endpoint 可以直接通信。Network 的实现可以是 Linux Bridge、VLAN 等。









22. ** Overlay 网络**

Docerk overlay 网络需要一个 key-value 数据库用于保存网络状态信息，包括 Network、Endpoint、IP 等。Consul、Etcd 和 ZooKeeper 都是 Docker 支持的 key-vlaue 软件，我们这里使用 Consul。

实验环境描述
直接使用docker-machine 创建的实验环境。在 docker 主机 host1（192.168.56.104）和 host2（192.168.56.105）上实践各种跨主机网络方案，在 192.168.56.101 上部署支持的组件，比如 Consul。

38. ** 以容器方式运行 Consul**
```
docker run -d -p 8500:8500 -h consul --name consul progrium/consul -server -bootstrap
```

容器启动后，可以通过 http://192.168.56.101:8500 访问 Consul
http://192.168.56.101:8500

39. ** docker daemon 的配置文件（所有docker宿主机）**

/etc/docker/daemon.json
```
  "hosts":["tcp://0.0.0.0:2376","unix:///var/run/docker.sock"],
  "cluster-store": "consul://10.0.0.11:8500",
  "cluster-advertise": "10.0.0.11:2376"
```




40. ** 重启docker**
```
systemctl daemon-reload 
systemctl restart docker.service
```




41. ** 创建 overlay 网络**

在 host1 中创建 overlay 网络 my_net
-d, --driver string        Driver to manage the Network (default "bridge")
docker network create -d overlay --subnet 192.168.1.0/24 --gateway 192.168.1.1 my_net

```
[root@host1 ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
6378931a146e        bridge              bridge              local
de423257940d        docker_gwbridge     bridge              local
f2e88b05bee1        host                host                local
32b44631dc90        my_net              overlay             global
077778bf9572        none                null                local
f5c13acdbe14        ov_net1             overlay             global
```

```
[root@host2 ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
a2b4299b2113        bridge              bridge              local
39a03dbfaf39        docker_gwbridge     bridge              local
78a1f06bd411        host                host                local
32b44631dc90        my_net              overlay             global
c0946fb465ae        none                null                local
f5c13acdbe14        ov_net1             overlay             global

docker network inspect my_net 
[
  ......
            "Config": [
                {
                    "Subnet": "192.168.1.0/24",
                    "Gateway": "192.168.1.1"
                }
            ]
        },
......
```

42. ** 运行容器并连接到my_net**

host1
```
docker run -d --privileged --name web1 --network my_net centos:latest /usr/sbin/init
```

```
docker inspect web1 |grep -i ipaddr
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "192.168.1.2",
```

host2
```
docker run -d --privileged --name web2 --network my_net centos:latest /usr/sbin/init
```

```
docker inspect web2 |grep -i ipaddr
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "192.168.1.3",
```



23. ** macvlan 网络**


---

如果网络不通可以设置：
打开网卡的混杂模式
```
ip link set eth0 promisc on
```
开启内核转发
```
net.ipv4.ip_forward = 1
```


---



创建macvlan网络（可以使用宿主机的网络）
```
docker network create --driver macvlan --subnet 10.0.0.0/24 --gateway 10.0.0.254 -o parent=eth0 macvlan_1
```

所有宿主机都创建相同的macvlan网络

创建容器
```
docker run -it --network macvlan_1 --ip=10.0.0.3 busybox:latest /bin/sh
```


host1
centos7:
```
docker run -d --privileged --name test1 --ip=10.0.0.8 --network macnet centos:latest /usr/sbin/init
```
centos6:
```
docker run -d --privileged --network macnet --ip 10.0.0.10 --name oldboy -h oldboy centos6-ssh:latest /sbin/init
```

host2
centos7:
```
docker run -d --privileged --name test2 --ip=10.0.0.7 --network macnet centos:latest /usr/sbin/init
```
centos6:
```
docker run -d --privileged --network macnet --ip 10.0.0.11 --name oldboy1 -h oldboy1 centos6-ssh:latest /sbin/init
```




24. ** docker网络桥接**
43. ** 停止docker服务 **
```
/etc/init.d/docker stop
systemctl stop docker.service
```


44. ** 停止docker0网卡**
```
ifconfig docker0 down
ip link set dev docker0 down
```


45. ** 删除默认的网络docker0**
```
brctl delbr docker0
```

46. ** 修改docker配置文件**
```
vim /etc/sysconfig/docker
OPTIONS='--selinux-enabled  改为
OPTIONS='--selinux-enabled -b=br0
```

47. ** 启动docker服务**
```
systemctl start docker.service
```

48. ** 安装pipework**
```
git clone https://github.com/jpetazzo/pipework
cp ~/pipework/pipework /usr/local/bin/
```


49. ** 启动一个手动设置网络的容器**

这里最好不要让docker自动获取ip，下次启动会有变化而且自动获取的ip可能会和物理网段中的ip冲突
```
docker run -d --net=none --privileged=true -v /sys/fs/cgroup:/sys/fs/cgroup --name web01 tutum/centos /usr/sbin/init
```

```
docker run -dit --net=none --privileged=true -v /sys/fs/cgroup:/sys/fs/cgroup --name mycentos6 centos:6 /bin/bash
```

50. ** 为web01容器设置一个与桥接物理网络同地址段的ip@网关**
```
pipework br0 web01 10.0.0.20/24@10.0.0.254
```

51. ** 进入容器查看ip**

```
docker exec -it web01 /bin/bash
```









25. ** 容器卷容器**

挂载目录 -v
```
docker run -d --name nginx -p 80:80 -v /data:/usr/share/nginx/html nginx:latest
```

--volumes-from
```
docker run -d --name nginx2 -p 81:80 --volumes-from nginx nginx:latest
```


```
docker create --name v_container -v ~/htdocs:/usr/local/apache2/htdocs -v /var/tools/ busybox
docker run --name web1 -d -p 80 --volumes-from v_container httpd
```


# **Dockerfile**
https://github.com/docker-library

用于制作docker镜像

[root@db01 ~]# ll /opt/mynginx/
总用量 8
-rw-r--r-- 1 root root 279 9月   3 01:29 Dockerfile
-rw-r--r-- 1 root root  41 9月   3 01:25 index.html

52. ** 创建dockerfile**

vim Dockerfile
```
FROM scratch
COPY hello /
CMD ["/hello"]
```


vim Dockerfile
```
FROM centos
RUN yum -y install httpd
```

```
docker build -t centos-httpd .
```



vim Dockerfile
```
FROM centos
MAINTAINER lyl admin@aclstack.com
RUN rpm -ivh http://mirrors.aliyun.com/epel/7/x86_64/e/epel-release-7-10.noarch.rpm && yum install nginx -y && echo "daemon off;" >> /etc/nginx/nginx.conf
ADD index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx"]
```



```
ls /opt/centos6-base/
Dockerfile  rootfs.tar.xz
```

```
Dockerfile
cat /opt/centos6-base/Dockerfile 
FROM scratch
ADD rootfs.tar.xz /
CMD ["/bin/bash"]
```


53. ** 创建镜像**
```
[root@db01 centos6-base]# pwd
/opt/centos6-base
[root@db01 centos6-base]# ls
Dockerfile  rootfs.tar.xz
```


```
docker image build -t centos6-base .
```






54. ** 附1**
```
[root@db01 centos6-ssh]# pwd
/opt/centos6-ssh
```

```
[root@db01 centos6-ssh]# ls
Dockerfile  init.sh  rootfs.tar.xz
```

[root@db01 centos6-ssh]# cat Dockerfile 
```
FROM scratch
ADD rootfs.tar.xz /
RUN yum install openssh-server httpd -y
ADD init.sh /init.sh
CMD ["/bin/bash","/init.sh"]
```


[root@db01 centos6-ssh]# cat init.sh 
```
#!/bin/bash
/etc/init.d/httpd start
/etc/init.d/sshd start
echo 123456 |passwd --stdin root
tail -f /var/log/httpd/access_log
```

```
docker image build -t centos6-ssh-http .
```

```
docker run -d -p 1122:22 -p 80:80 --name web01 centos6-ssh-http:latest
```


55. ** 附2  centos6制作**
```
wget https://mirrors.tuna.tsinghua.edu.cn/lxc-images/images/centos/6/amd64/default/20180330_02%3A16/rootfs.tar.xz
```

```
[root@mycentos centos6-ssh]# pwd
/opt/centos6-ssh
[root@mycentos centos6-ssh]# ls
Dockerfile  init.sh  rootfs.tar.xz
```

脚本
[root@mycentos centos6-ssh]# cat init.sh 
```
#!/bin/bash
echo 123456 |passwd --stdin root
/etc/init.d/sshd start
tail -f /var/log/messages
```

Dockerfile
[root@mycentos centos6-ssh]# cat Dockerfile 
```
FROM scratch
ADD rootfs.tar.xz /
RUN curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
RUN yum install openssh-server -y
ADD init.sh /init.sh
CMD ["/bin/bash","/init.sh"]
```

制作镜像
```
docker image build -t centos6 /opt/centos6-ssh/
```





56. ** 附3  centos7制作**
```
wget https://mirrors.tuna.tsinghua.edu.cn/lxc-images/images/centos/7/amd64/default/20180330_02%3A16/rootfs.tar.xz
```

```
[root@mycentos centos7-ssh]# pwd
/opt/centos7-ssh
[root@mycentos centos7-ssh]# ls
Dockerfile  init.sh  rootfs.tar.xz
```

cat Dockerfile 
```
FROM scratch
ADD rootfs.tar.xz /
RUN curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
RUN echo 123456 |passwd --stdin root
RUN yum install openssh-server -y
RUN systemctl enable sshd
ADD init.sh /init.sh
CMD ["/bin/bash","/init.sh"]
```

cat init.sh 
```
#!/bin/bash
/usr/sbin/init
tail -f /var/log/messages
```


制作镜像
```
docker image build -t centos7 /opt/centos7-ssh/
```


启动容器
```
docker run -d --privileged --name test centos7:latest /usr/sbin/init
```







# **Dockerfile语法**
18. ** FROM**

FROM ImageName
Dockerfile的第一句必须是FROM

19. ** MAINTAINER**

MAINTAINER wuxingge 1226032602@qq.com

20. ** RUN**

RUN yum install -y openssh-server && yum clean all
shell中的运行命令


21. ** CMD**

格式1：CMD ["executable","param1","param2"]#运行一个可执行的文件并提供参数。
CMD ["/usr/bin/supervisord"]

格式2：CMD ["param1","param2"] #为ENTRYPOINT指定参数。
每个dockerfile只能有一条CMD命令，如果多条，执行最后的一条，是在docker容器启动的的时候执行的命令，如果在启动时指定，会替代CMD命令。

22. ** ENTRYPOINT**

格式： ENTRYPOINT ["executable", "param1","param2"]
ENTRYPOINT ["/usr/bin/nginx"]
#和CMD类似都是配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖，同样，每个 Dockerfile 中只能有一个ENTRYPOINT，当指定多个时，只有最后一个起效。
#但ENTRYPOINT没有CMD的可替换特性，也就是你启动容器的时候增加运行的命令不会覆盖ENTRYPOINT指定的命令。

23. ** EXPOSE**

格式：EXPOSE prot ...
example：EXPOSE 80 9000
#设置容器的的端口，外部可使用-p指定映射，或者-P随机端口映射。

24. ** ENV**

格式：ENV key=value
ENV JAVA_HOME=/usr/local/src/jdk
#设置环境变量。

25. ** ADD**

格式：ADD <src> <dest>
ADD epel-release-latest-7.noarch.rpm /tmp/
#将当前目录的指定文件拷贝到容器的中，如果是可识别的压缩文件，docker会自动解压

26. ** COPY**

把宿主机的文件拷贝到镜像中



27. ** WORKDIR**

格式：WORKDIR /path
WORKDIR /tools/
#相当于cd命令，多个是相对路径，避免混淆，建议使用绝对路径

28. ** VOLUME**

格式：VOLUME ["/data"]
VOLUME ["/data/lnp"]
#可以将本地的文件夹或者其他容器的文件挂载到此容器中。






26. **自定义docker镜像**
57. ** 使用已有的容器生成镜像**

```
docker run -dit centos:7
```


```
[root@centos72 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
2c6bb22974a1        centos:6            "/bin/bash"         37 minutes ago      Up 27 seconds                           mydocker
```

进入容器

安装httpd
```
yum install httpd -y
chkconfig httpd on  # 或centos7   systemctl enable httpd
exit
```

提交镜像
```
[root@centos72 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
2c6bb22974a1        centos:6            "/bin/bash"         54 minutes ago      Exited (0) 2 minutes ago                       mydocker
[root@centos72 ~]# docker commit -m "centos http  server" 2c6bb22974a1  xdz/httpd:v1                
sha256:923d30997fc3cc4825a0ab8ac1a10e49fcc066f98af13395591a7f9fe4270627
```



58. ** 新提交的镜像启动docker容器**
```
docker run -dit -p 80:80 xdz/httpd:v1 /sbin/init
```

‘-p’:端口映射，第一个80为本地端口，第二个80为docker容器端口 
xdz/httpd:v1:刚才提交的镜像名称 
/sbin/init:启动init程序，由它去进行chkconfig http on的操作 

正常情况下访问本机的ip即可访问docker容器中的http服务器

```
[root@centos72 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS                NAMES
0cb34cb82c20        xdz/httpd:v1        "/sbin/init"        35 seconds ago      Up 33 seconds              0.0.0.0:80->80/tcp   adoring_kirch
2c6bb22974a1        centos:6            "/bin/bash"         About an hour ago   Exited (0) 9 minutes ago                        mydocker
```


```
docker run -d -p 2022:22 --name centos6 centos6-ssh /usr/sbin/sshd -D
```

cat init.sh 
```
#!/bin/bash
/etc/init.d/httpd start
/usr/sbin/sshd -D
```

```
docker run -d --name centos6-httpd -p 3022:22 -p 80:80 centos6-httpd:latest /bin/bash /init.sh
```


27. ** 私有仓库**
59. ** 普通的registry**
```
vim /usr/lib/systemd/system/docker.service
...
ExecStart=/usr/bin/dockerd --insecure-registry 10.0.0.11:5000
```



```
docker run -d -p 5000:5000 --restart=always --name registry -v /opt/myregistry:/var/lib/registry  registry
```



60. ** 带basic认证的registry**
```
mkdir /opt/registry-var/auth/ -p
yum install httpd-tools -y
htpasswd  -Bbn oldboy 123456  >> /opt/registry-var/auth/htpasswd
```


```
docker run -d -p 5000:5000 --restart=always --name registry -v /opt/registry-var/auth/:/auth/ -v /opt/myregistry:/var/lib/registry -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd registry
```


配置文件
cat /etc/docker/daemon.json
```
{
  "registry-mirrors": ["https://gqk8w9va.mirror.aliyuncs.com"],
  "graph":"/opt/mydocker",
  "insecure-registries":["10.0.0.11:5000"]
}
```


登录
```
docker login 10.0.0.11:5000
```

浏览器访问
http://registry.wuxingge.org:5000/v2/


61. ** 镜像做标记(tag)**

使用 docker tag 将 tutum/centos:latest  这个镜像标记为 47.104.108.63:5000/wuxingge/centos:latest
```
docker tag  IMAGE[:TAG]  [REGISTRYHOST/] [USERNAME/]NAME[:TAG] 
```


```
docker tag tutum/centos:latest 47.104.108.63:5000/wuxingge/centos:latest
```


62. ** 客户端及服务器修改dokcer配置文件并重启docker（docker-ee配置）**

cat /etc/sysconfig/docker
OPTIONS='--selinux-enabled --insecure-registry 47.104.108.63:5000 --log-driver=journald --signature-verification=false'


63. ** 镜像推入镜像仓库**
```
docker push 47.104.108.63:5000/wuxingge/centos:latest
```



64. ** 查看docker仓库中的镜像**
```
curl http://47.104.108.63:5000/v2/_catalog
```
{"repositories":["wuxingge/centos"]}

```
curl -u oldboy:123456 http://10.0.0.11:5000/v2/_catalog
```
{"repositories":["wuxingge/alpine"]}


65. ** docker仓库镜像存放位置**
```
/data/registry/docker/registry/v2/repositories/wuxingge/
```


66. ** 拉镜像到本地**

```
docker pull 10.0.0.11:5000/wuxingge/alpine:latest
```



67. ** 远程访问配置**
```
cat /etc/sysconfig/docker-network 
# /etc/sysconfig/docker-network
DOCKER_NETWORK_OPTIONS="-H unix:///var/run/docker.sock -H 0.0.0.0:2375"
```

68. ** 重启docker**

systemctl daemon-reload 
systemctl restart docker.service

69. ** 远程访问**

docker -H 47.104.108.63:2375 images


28. ** Docker镜像仓库Harbor**

https://blog.csdn.net/aixiaoyang168/article/details/73549898


[https://github.com/goharbor/harbor/blob/master/docs/installation_guide.md](https://github.com/goharbor/harbor/blob/master/docs/installation_guide.md)



# docker hub
```
docker tag 10.0.0.11:5000/wuxingge/kubernetes-dashboard:v1.10.1 wuxingge/mydocker:kubernetes-dashboard-v1.10.1
```


```
docker login 
docker push wuxingge/mydocker:kubernetes-dashboard-v1.10.1
```



```
docker pull wuxingge/mydocker:kubernetes-dashboard-v1.10.1
```





29. **docker-compose **

用于启动容器
70. ** 准备**
```
docker run -d --name web1 nginx
docker run -d --name web2 nginx
```


```
cat haproxy.cfg 
global
  log 127.0.0.1 local0
  log 127.0.0.1 local1 notice
  maxconn 4096

defaults
  log global
  mode http
  option httplog
  option dontlognull
  timeout connect 5000ms
  timeout client 5000ms
  timeout server 5000ms

listen stats
  bind 0.0.0.0:1080
  mode http
  stats enable
  stats hide-version
  stats uri /stats
  stats auth admin:admin

frontend balance
  bind 0.0.0.0:80
  default_backend web_backends

backend web_backends
  mode http
  option forwardfor
  balance roundrobin 
  server web1 web1:80 check
  server web2 web2:80 check
```


```
docker run -it --rm -v /opt/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg --link web1 --link web2 -p 7777:1080 -p 80:80 haproxy
```

访问
http://10.0.0.201:7777





71. ** 安装使用docker-compose**
```
yum install python2-pip -y
pip install docker-compose
```

##验证安装成功
```
docker-compose -v
```

```
[root@db01 haproxy]# pwd
/opt/haproxy
[root@db01 haproxy]# cat docker-compose.yml 
web1:
  image: nginx
  expose:
    - 80

web2:
  image: nginx
  expose:
    - 80

haproxy:
  image: haproxy
  volumes:
    - /opt/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg
  links:
    - web1
    - web2
  ports:
    - "7777:1080"
    - "80:80"
```


```
docker-compose up
```


访问
http://10.0.0.201:7777


30. **Docker Compose 配置文件详解**

一份标准配置文件应该包含 version、services、networks 三大部分，其中最关键的就是 services 和 networks 两个部分
```
version: '3'
services:
  web:
    image: dockercloud/hello-world
    ports:
      - 8080
    networks:
      - front-tier
      - back-tier

  redis:
    image: redis
    links:
      - web
    networks:
      - back-tier

  lb:
    image: dockercloud/haproxy
    ports:
      - 80:80
    links:
      - web
    networks:
      - front-tier
      - back-tier
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock 

networks:
  front-tier:
    driver: bridge
  back-tier:
    driver: bridge
```


服务除了可以基于指定的镜像，还可以基于一份 Dockerfile，在使用 up 启动之时执行构建任务，这个构建标签就是 build，它可以指定 Dockerfile 所在目录。Compose 将会利用它自动构建这个镜像，然后使用这个镜像启动服务容器


build: ./dir
image: webapp:tag




docker-compose.yml
```
version: '3'
services:
  weba:
    build: ./web
    expose:
      - 80
  webb:
    build: ./web
    expose:
      - 80
  webc:
    build: ./web
    expose:
      - 80
  haproxy:
    image: haproxy:latest
    volumes:
      - ./haproxy:/haproxy-override
      - ./haproxy/haproxy.cfg:/usr/local/etc/haproxy/haproxy.c
  fg:ro
    links:
      - weba
      - webb
      - webc
    ports:
      - "80:80"
      - "70:70"
    expose:
      - "80"
      - "70"
```





```
compose-haproxy-web
├── docker-compose.yml
├── haproxy
│   └── haproxy.cfg
└── web
    ├── Dockerfile
    ├── index.html
    └── index.py
```


在该目录下执行 docker-compose up 命令，会整合输出所有容器的输出。


72. ** wordpress-compose**

cd my_wordpress/

vi docker-compose.yml
```
version: '3'
services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
volumes:
    db_data:
```


#启动
```
docker-compose up
```
#后台启动
```
docker-compose up -d
```





31. ** 附录**
```
docker pull nextcloud
docker run -d -p 88:80 --name cloud docker.io/nextcloud:latest
```

```
docker run -d -p 88:80 --restart=always -v /data/www/html:/var/www/html --name cloud docker.io/nextcloud:latest
```


docker文档
```
docker run -d -p 4000:4000 --restart=always --name dockerdoc docs/docker.github.io:latest
```
















