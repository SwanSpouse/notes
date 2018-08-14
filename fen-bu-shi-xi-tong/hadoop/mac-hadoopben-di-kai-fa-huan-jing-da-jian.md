### mac hadoop 本地开发环境搭建

暂时还没有搭建成功。这里记录一步步踩的坑。

首先要安装java，maven，然后我的maven是通过brew 安装的。brew 安装的maven会在Documents里面生成一个maven目录。这个很奇怪，不是统一在/usr/local/Celler/里面生成吗？

然后我修改了maven的中央仓库。用了公司的，同步maven依赖的时候没有啥问题，但是同步maven插件的时候报错了，然后我把maven中央库的地址移除了。现在好像好了。

mvn assembly: assembly 能够执行成功，这样就打成了一个jar包。但是没有hadoop环境，还无法执行这个jar包。

#### hadoop环境

我看brew里面有hadoop，我就先用brew install 了一下hadoop来试试。安了好长时间没有安装上。所以我就打算从官网下载一个hadoop包来试试。

按照https://www.cnblogs.com/fishbay/archive/2017/07/24/7229502.html 这里描述的步骤，已经启动了本地的NameNode, DataNode, NodeManager这些进程。而且50070的web页面也能够打开了。现在看看是不是本能够运行mapreduce程序。但是我记得好像本地运行mapreduce程序是不需要启动这些东西的。当时的那个开发环境搭建是另一套逻辑。

Exception in thread "main" java.io.IOException: Mkdirs failed to create /var/folders/zz/zyxvpxvq6csfxvn\_n0000000000000/T/hadoop-unjar7560830407203462827/META-INF/license

期间报这样一个错误，https://github.com/sozykin/mipr/issues/2 这个方法可以解决，但是还没有运行起来。说是连接不上本地的NameNode，但是我记得当时我们做的时候没有这么多事情，都不用启动NameNode直接就可以运行起来。

确实是环境的问题，在52的机器上面可以正常运行，但是在本地却无法正常执行。  


