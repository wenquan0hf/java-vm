# Java 代码编译和执行的整个过程

Java 代码编译是由 Java 源码编译器来完成，流程图如下所示：

![](images/javadebug.gif)

Java 字节码的执行是由 JVM 执行引擎来完成，流程图如下所示：

![](images/jvmdebug.gif)

Java 代码编译和执行的整个过程包含了以下三个重要的机制：

- Java 源码编译机制
- 类加载机制
- 类执行机制

## Java 源码编译机制

Java 源码编译由以下三个过程组成：

- 分析和输入到符号表
- 注解处理
- 语义分析和生成 class 文件

流程图如下所示：

![](images/workflow.gif)


最后生成的 class 文件由以下部分组成：

- 结构信息。包括 class 文件格式版本号及各部分的数量与大小的信息。
- 元数据。对应于 Java 源码中声明与常量的信息。包含类/继承的超类/实现的接口的声明信息、域与方法声明信息和常量池。
- 方法信息。对应 Java 源码中语句和表达式对应的信息。包含字节码、异常处理器表、求值栈与局部变量区大小、求值栈的类型记录、调试符号信息。

## 类加载机制

JVM 的类加载是通过 ClassLoader 及其子类来完成的，类的层次关系和加载顺序可以由下图来描述：

![](images/jvmclass.gif)

**1）Bootstrap ClassLoader**

负责加载`$JAVA_HOME中jre/lib/rt.jar`里所有的 class，由 C++ 实现，不是 ClassLoader 子类。

**2）Extension ClassLoader**

负责加载Java平台中扩展功能的一些 jar 包，包括`$JAVA_HOME中jre/lib/*.jar`或`-Djava.ext.dirs`指定目录下的 jar 包。

**3）App ClassLoader**

负责记载 classpath 中指定的 jar 包及目录中 class。

**4）Custom ClassLoader**

属于应用程序根据自身需要自定义的 ClassLoader，如 Tomcat、jboss 都会根据 J2EE 规范自行实现 ClassLoader。

加载过程中会先检查类是否被已加载，检查顺序是自底向上，从 Custom ClassLoader 到 BootStrap ClassLoader 逐层检查，只要某个 Classloader 已加载就视为已加载此类，保证此类只所有 ClassLoade r加载一次。而加载的顺序是自顶向下，也就是由上层来逐层尝试加载此类。

## 类执行机制

JVM 是基于栈的体系结构来执行 class 字节码的。线程创建后，都会产生程序计数器（PC）和栈（Stack），程序计数器存放下一条要执行的指令在方法内的偏移量，栈中存放一个个栈帧，每个栈帧对应着每个方法的每次调用，而栈帧又是有局部变量区和操作数栈两部分组成，局部变量区用于存放方法中的局部变量和参数，操作数栈中用于存放方法执行过程中产生的中间结果。栈的结构如下图所示：

![](images/classrun.gif)
