# lex&yacc

[Lex & Yacc](http://dinosaur.compilertools.net/)是用来生成词法分析器和语法分析器的工具，它们的出现简化了编译器的编写。`Lex & Yacc`分别是由贝尔实验室的[Mike Lesk](https://en.wikipedia.org/wiki/Mike_Lesk)和[Stephen C. Johnson](https://en.wikipedia.org/wiki/Stephen_C._Johnson)在 1975 年发布。对于 Java 程序员来说，更熟悉的是[ANTLR](http://www.antlr.org/)，`ANTLR 4`提供了`Listener`+`Visitor`组合接口， 不需要在语法定义中嵌入`actions`，使应用代码和语法定义解耦。`Spark`的 SQL 解析就是使用了`ANTLR`。

一般编程语言的语法处理，都会有以下的过程。

* 词法分析： 将源代码分割成若干个记号的处理。
* 语法分析：即从记号构建分析树的处理。构建AST树，分析树也叫作语法树或抽象语法树。
* 语义分析：经过语法分析生成的分析树，并不包含数据类型等语义信息。因此在语义分析阶段，会检查程序中是否含有语法正确但是存在逻辑问题的错误。
* 生成代码：如果是C语言等生成机器码的编译器或Java这样生成字节码的编译器，在分析树构建完毕后会进入代码生成阶段。

执行词法分析的程序称为词法分析器。lex的工作就是根据词法规则自动生成词法分析器。

执行语法分析的程序则称为解析器。yacc就是根据语法规则自动生成解析器的程序。

## reference

* [https://www.pingcap.com/blog-cn/tidb-source-code-reading-5/](https://www.pingcap.com/blog-cn/tidb-source-code-reading-5/)

