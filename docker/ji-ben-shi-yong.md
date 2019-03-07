### Docker基本使用

#### 获取镜像

可以使用docker pull 直接从Docker Hub镜像源来下载镜像。

命令格式为：docker pull NAME\[:TAG\]

* 其中NAME 是镜像仓库的名称（用来区分镜像），TAG是镜像的标签（往往用来表示版本信息）
* e.g. docker pull ubuntu:14.04
* 如果不显式的指定TAG，则默认会选择latest标签，这个会下载最新版本的镜像。

下载的过程中可以看出，镜像文件一般由若干层\(layer\)组成，6c953ac5d795这样串是层的唯一ID。使用Docker pull命令下载时会获取并输出镜像的各层信息。当不同的镜像包括相同的层时，本地仅存储层的一份内容，减小了需要的存储空间。

docker pull ubuntu:14.04命令相当于docker pull registry.hub.docker.com/ubuntu:14.04 命令，即从默认的注册服务器Docker Hub Registry中的Ubuntu 仓库来下载标记为14.04的镜像。

如果从非官方的仓库下载，则需要在仓库名称前指定完整的仓库地址。例如从网易蜂巢的镜像源来下载ubuntu14.04镜像，可以使用如下命令。

docker pull hub.c.163.com/public/ubuntu:14.04

![](/assets/docker images.png)

* REPOSITORY:  来自于哪个仓库，ubuntu仓库用来保存ubuntu系列的基础镜像。

* 镜像的标签信息：比如14.04 、latest用来标注不同的版本信息，标签只是标记。并不能标识镜像内容。

* IMAGE ID： 唯一识别标识。

* 创建时间：镜像的最后更新时间。

* 镜像大小：

**使用tag命令添加镜像标签**

docker tag ubuntu:latest myubuntu:latest

**使用inspect命令查看该镜像的详细信息**

docker inspect ubuntu:14.04

#### 查看镜像、搜寻镜像、删除镜像、创建镜像、存出和载入镜像、上传镜像

相应的命令比较简单。用的时候再查就可以了。

使用docker save和 docker load实现存出和载入。

#### 启动容器

使用docker create 来创建一个容器，创建出的新容器处于停止状态。

使用docker start来启动一个已经创建的容器。

docker run = docker create + docker start，创建并启动一个容器。

docker run 背后的操作:

* 检查本地是否存在指定的镜像，不存在就从公有仓库下载；
* 利用镜像创建一个容器，并启动该容器；
* 分配一个文件系统给容器，并在只读的镜像层外面挂载一层可读可写层；
* 从宿主主机的配置的网桥接口中桥接一个虚拟接口到容器中；
* 从网桥的地址池配置一个IP地址给容器；
* 执行用户指定的应用程序；

docker run -it ubuntu:14.04 /bin/bash 其中-t选项让Docker分配一个伪终端（psedo-tty）并绑定到容器的标准输入上，-i则让容器的标准输入保持打开。

#### 进入容器

docker从1.3.0版本提供了exec命令，可以在容器内直接执行任意命令。

docker exec -it 243c32545da7 /bin/bash

