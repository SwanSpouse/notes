# 单例模式（Singleton Pattern）

#### **定义：**

Ensure a class has only one instance, and provide a global point of access to it.（确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例。）

#### 通用代码：

```java
public class Singleton {
     private static final Singleton singleton = new Singleton();
     //限制产生多个对象
     private Singleton(){
     }
     //通过该方法获得实例对象
     public static Singleton getSingleton(){
             return singleton;
     }  
     //类中其他方法，尽量是static
     public static void doSomething(){
     }
}
```

#### 使用场景：

● 要求生成唯一序列号的环境；

● 在整个项目中需要一个共享访问点或共享数据，例如一个Web页面上的计数器，可以不用把每次刷新都记录到数据库中，使用单例模式保持计数器的值，并确保是线程安全的；

● 创建一个对象需要消耗的资源过多，如要访问IO和数据库等资源；

● 需要定义大量的静态常量和静态方法（如工具类）的环境，可以采用单例模式（当然，也可以直接声明为static的方式）。



#### 参考

* https://www.cnblogs.com/tongkey/p/7170826.html



