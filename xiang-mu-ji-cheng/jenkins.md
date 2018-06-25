### Jenkins

#### Jenkins概念:

Jenkins是一个功能强大的应用程序，允许持续集成和持续交付项目，无论用的是什么平台。这是一个免费的源代码，可以处理任何类型的构建或持续集成。集成Jenkins可以用于一些测试和部署技术。Jenkins是一种软件允许持续集成。

典型的工作流包括以下几个步骤

1. 开发
2. 提交
3. 编译
4. 测试
5. 发布

有了Jenkins的帮助，在这5步中，除了第1、2步，后续的3步都是自动化完成的。当你完成了开发和提交代码，Jenkins会自动运行你的编译脚本，编译成功后，再运行你的测试脚本，这一步成功后，接着它会帮你把新程序发布出去。

#### Jenkins目的：

1、持续、自动地构建/测试软件项目。

2、监控软件开放流程，快速问题定位及处理，提示开放效率。

#### 特性：

开源的java语言开发持续集成工具，支持CI，CD。

易于安装部署配置：可通过yum安装,或下载war包以及通过docker容器等快速实现安装部署，可方便web界面配置管理。

消息通知及测试报告：集成RSS/E-mail通过RSS发布构建结果或当构建完成时通过e-mail通知，生成JUnit/TestNG测试报告。

分布式构建：支持Jenkins能够让多台计算机一起构建/测试。

文件识别:Jenkins能够跟踪哪次构建生成哪些jar，哪次构建使用哪个版本的jar等。

丰富的插件支持:支持扩展插件，你可以开发适合自己团队使用的工具，如git，svn，maven，docker等。

#### 发布流程：

产品设计成型 -&gt; 开发人员开发代码 -&gt; 测试人员测试功能 -&gt; 运维人员发布上线

持续集成 （Continuous integration，简称CI）

持续交付（Continuous delivery）

持续部署（continuous deployment）

![](/assets/jenkins持续部署.png)

#### jenkins pipeline

如果想每次`git commit`时自动执行该pipeline，可以让Jenkins对git进行轮询，每分钟检查git仓库有没有更新。

设置完毕后，一旦你的git仓库收到新的提交，就会触发这个pipeline的运行。

  
  


  




