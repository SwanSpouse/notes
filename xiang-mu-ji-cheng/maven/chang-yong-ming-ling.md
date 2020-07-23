# 常用命令

编译源代码

* mvn compile

编译测试代码

* mvn test-compile

运行测试

* mvn test

只打jar包

* mvn jar:jar

只测试不编译，也不测试编译:

* mvn -test -skipping compile -skipping test-compile

查看当前项目已被解析的依赖

* mvn dependency:list

生成target目录，编译、测试代码，生成测试报告，生成jar/war文件

* mvn package

想要查看完整的依赖踪迹，包含那些因为冲突或者其它原因而被拒绝引入的构件，打开 Maven 的调试标记运行 :

* mvn -install -X

给任何目标添加maven.test.skip 属性就能跳过测试 :

* mvn install -Dmaven.test.skip=true

构建装配Maven Assembly 插件是一个用来创建你应用程序特有分发包的插件 :

* mvn install assembly:assembly

打印出已解决依赖的列表:

* mvn dependency:resolve

打印整个依赖树 :

* mvn dependency:tree

