# 单例模式

## **定义：**

Ensure a class has only one instance, and provide a global point of access to it.（确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例。）

单例的实现主要是通过以下两个步骤：

1. **将构造方法定义为私有方法，**这样其他代码就无法通过调用该类的构造方法来实例化该类的对象，只有通过该类提供的静态方法来得到该类的唯一实例；
2. **在该类内提供一个静态方法，**当调用这个方法时，如果类持有的引用不为空就返回这个引用；如果类持有的引用为空就创建该类的实例并将实例的引用赋值给类持有的引用。

## 单例五种写法

### 饿汉模式

优点：这种写法比较简单，就是在类装载的时候就完成实例化。避免了线程同步问题。  
缺点：在类装载的时候就完成实例化，没有达到Lazy Loading的效果。如果从始至终从未使用过这个实例，则会造成内存的浪费。

```java
public class Singleton {

    private final static Singleton INSTANCE = new Singleton();

    private Singleton(){}

    public static Singleton getInstance(){
        return INSTANCE;
    }
}
```

### 懒汉模式

这种写法起到了Lazy Loading的效果，但是只能在单线程下使用。如果在多线程下，一个线程进入了if \(singleton == null\)判断语句块，还未来得及往下执行，另一个线程也通过了这个判断语句，这时便会产生多个实例。所以在多线程环境下不可使用这种方式。

```java
public class Singleton {

    private static Singleton singleton;

    private Singleton() {}

    public static Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }
}
```

### 懒汉模式 + 线程安全、同步方法

解决上面第三种实现方式的线程不安全问题，做个线程同步就可以了，于是就对getInstance\(\)方法进行了线程同步。

缺点：效率太低了，每个线程在想获得类的实例时候，执行getInstance\(\)方法都要进行同步。而其实这个方法只执行一次实例化代码就够了，后面的想获得该类实例，直接return就行了。方法进行同步效率太低要改进。

```java
public class Singleton {

    private static Singleton singleton;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }
}
```

### 懒汉模式 + 双重检查

Double-Check，如代码中所示，进行了两次if \(singleton == null\)检查，这样就可以保证线程安全了。这样，实例化代码只用执行一次，后面再次访问时，判断if \(singleton == null\)，直接return实例化对象。

```java
public class Singleton {

    private static volatile Singleton singleton;

    private Singleton() {}

    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

### 静态内部类

这种方式跟饿汉式方式采用的机制类似，但又有不同。两者都是采用了类装载的机制来保证初始化实例时只有一个线程。不同的地方在饿汉式方式是只要Singleton类被装载就会实例化，没有Lazy-Loading的作用，而静态内部类方式在Singleton类被装载时并不会立即实例化，而是在需要实例化时，调用getInstance方法，才会装载SingletonInstance类，从而完成Singleton的实例化。

类的静态属性只会在第一次加载类的时候初始化，所以在这里，JVM帮助我们保证了线程的安全性，在类进行初始化时，别的线程是无法进入的。

优点：避免了线程不安全，延迟加载，效率高。

```java
public class Singleton {

    private Singleton() {}

    private static class SingletonInstance {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonInstance.INSTANCE;
    }
}
```

## 参考

* [https://www.cnblogs.com/tongkey/p/7170826.html](https://www.cnblogs.com/tongkey/p/7170826.html)
* [http://www.runoob.com/design-pattern/singleton-pattern.html](http://www.runoob.com/design-pattern/singleton-pattern.html)
* [https://www.cnblogs.com/zhaoyan001/p/6365064.html](https://www.cnblogs.com/zhaoyan001/p/6365064.html)

