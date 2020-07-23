# Java 内存模型

这里主要对java内存模型进行介绍。主要包括：

* 知道volatile怎么用，Atomic类的用法
* CAS是什么；能写出用double check locking实现的线程安全的单例
* 了解happen-before规则（能利用此规则分析代码是否线程安全）
* 了解Java里的类是怎么保存的。Object的头部都有什么。什么是false-sharing，最好举例，如果研究过LMAX的disruptor更好

