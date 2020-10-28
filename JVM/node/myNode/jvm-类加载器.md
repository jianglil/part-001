三层类加载器及其父子关系

BootstrapClassLoader

ExtClassLoader

AppClassLoader

父子关系链





详解启动类加载器

没有实体，只是那一段逻辑叫做启动类加载器

什么是双亲委派

打破双亲委派

SPI

为什么SPI打破了双亲委派？

==双亲委派也是java程序控制的，打破双亲委派也是换了一种实现逻辑==

SPI在数据库驱动中有应用

在JDBC4.0之后支持SPI方式加载java.sql.Driver的实现类。SPI实现方式为，通过ServiceLoader.load(Driver.class)方法，去各自实现Driver接口的lib的META-INF/services/java.sql.Driver文件里找到实现类的名字，通过Thread.currentThread().getContextClassLoader()类加载器加载实现类并返回实例。

#### 驱动加载的过程大致如上，那么是在什么地方打破了双亲委派模型呢？

先看下如果不用Thread.currentThread().getContextClassLoader()加载器加载，整个流程会怎么样。

1. 从META-INF/services/java.sql.Driver文件得到实现类名字DriverA
2. Class.forName("xx.xx.DriverA")来加载实现类
3. Class.forName()方法默认使用当前类的ClassLoader，JDBC是在DriverManager类里调用Driver的，当前类也就是DriverManager，它的加载器是BootstrapClassLoader。
4. 用BootstrapClassLoader去加载非rt.jar包里的类xx.xx.DriverA，就会找不到
5. 要加载xx.xx.DriverA需要用到AppClassLoader或其他自定义ClassLoader
6. 最终矛盾出现在，要在BootstrapClassLoader加载的类里，调用AppClassLoader去加载实现类

#### 这样就出现了一个问题：如何在父加载器加载的类中，去调用子加载器去加载类？

1. jdk提供了两种方式，Thread.currentThread().getContextClassLoader()和ClassLoader.getSystemClassLoader()一般都指向AppClassLoader，他们能加载classpath中的类
2. SPI则用Thread.currentThread().getContextClassLoader()来加载实现类，实现在核心包里的基础类调用用户代码

[SPI底层原理](https://zhuanlan.zhihu.com/p/257122662)

自定义类加载器

jvm中的沙箱安全机制