### Docker数据管理

#### 容器中管理数据的主要方式

数据卷\(Data Volumes\): 容器内数据直接映射到本地主机环境。

数据卷容器\(Data Volumne Containers\)：使用特定的容器维护数据卷。

#### 数据卷

数据卷是一个可供容器使用的特殊目录，它将主机操作系统目录直接映射进容器，类似于Linux系统的Mount操作。

数据卷可以提供很多有用的特性

* 数据卷可以在容器之间共享和重用，容器间传递数据将变得高效方便；
* 对数据卷内数据的修改会立刻生效，无论是容器内操作还是容器外操作；
* 对数据卷的更新不会影响镜像，解耦了应用和数据；
* 卷会一直存在，直到没有容器使用，可以安全的卸载它。

使用docker run命令的时候，使用-v标记可以在容器内创建一个数据卷

docker run -d -P --name web -v /webapp training/webapp python app.py

上面使用training/webapp镜像创建一个web容器，并创建一个数据卷挂载到容器的/webapp目录

docker run -d -P --name web -v /src/webapp:/opt/webapp training/webapp python app.py

上面的命令加载主机的/src/webapp目录到容器的/opt/webapp目录。通过上述命令，可以将一些程序或数据放到本地目录中，然后在容器内运行和使用。

### Docker端口映射

#### 端口映射实现访问容器

当容器中运行一些网络应用，要让外部访问这些应用时，可以通过—P或者-p参数来指定端口映射。当使用-P标记时，Docker会随机映射一个49000~49900的端口到内部容器开放的网络端口。

使用HostPort:ContainerPort格式将本地的5000端口映射到容器的5000端口

docker run -d -p 5000:5000 training/webapp python app.py

#### 自定义容器命名

使用--name 标记可以为容器自定义命名。这个名称需要是唯一的。

### reference :

* 《Docker技术入门与实战》





