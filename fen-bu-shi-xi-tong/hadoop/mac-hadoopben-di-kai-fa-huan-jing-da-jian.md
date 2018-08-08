### mac hadoop 本地开发环境搭建

暂时还没有搭建成功。这里记录一步步踩的坑。

首先要安装java，maven，然后我的maven是通过brew 安装的。brew 安装的maven会在Documents里面生成一个maven目录。这个很奇怪，不是统一在/usr/local/Celler/里面生成吗？

然后我修改了maven的中央仓库。用了公司的，同步maven依赖的时候没有啥问题，但是同步maven插件的时候报错了，然后我把maven中央库的地址移除了。现在好像好了。

mvn assembly: assembly 能够执行成功，这样就打成了一个jar包。但是没有hadoop环境，还无法执行这个jar包。

#### hadoop环境

我看brew里面有hadoop，我就先用brew install 了一下hadoop来试试。



