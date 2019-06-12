
## 单件模式（Singleton Pattern）
　　单件模式，确保一个类只有一个实例，并提供一个全局访问点。<br />
　　有一些对象其实只需要一个，比如常用于管理共享资源（数据库连接池、线程池）、缓存、对话框、处理偏好设置和注册表的对象、日志对象，这类对象只能有一个实例，多个实例会出问题。<br />
　　利用单件模式，可以在需要时才创建对象。而将对象赋值给全局变量，需要程序开始就创建好，如果没用到该对象，则会一直占用浪费资源。
  
### 单件模式的实现
　　将构造器声明为私有，实例化使用静态方法 getInstance() 来获取，会判断该对象是否已经实例过，如已经存在，则直接返回，保证只有一个实例对象。

```java
public class Singleton {
    // 利用静态变量来记录 Singleton 类的唯一实例
    private static Singleton uniqueInstance;
    
    // 把构造器声明为私有，只有自 Singleton 类才可以调用构造器
    private Singleton() { }
    
    // 用 getInstance() 方法实例化这个对象，并返回这个实例。如果不存在，则使用
    // 私有构造器产生一个 Singleton 实例，并把它赋值到静态变量 uniqueInstance 
    // 中。如果不需要这个实例，它就永远不会产生，即延迟实例化
    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }
        // 如果不为空，表示这个实例已经创建过了，直接返回
        return uniqueInstance;
    }
}
```

　　单件模式的类图如下：

![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_5/chapter_5_p1.png)

### 多线程下的单件模式
　　上面的代码存在缺陷，在多线程时会获得多个相同的实例，解决方法是加上同步关键字 synchronized，public static synchronized Singleton getInstance() ，这样就不会有多个线程进入这个方法。但同步关键字会造成性能下降，且只有第一次执行该方法时，才需要同步创建对象，后面已经存在对象，就不需要同步这个方法。<br />
　　优化方法如下：
  
- 不用延迟实例化，而是“急切”创建实例。如果应用程序总是创建并使用单件实例，或者在创建和运行时方面的负担不太繁重，可在程序运行时就创建。如下，JVM 在加载类时马上创建唯一的单件实例，保证在线程访问 uniqueInstance 之前；

```java
public class Singleton {
    // 在静态初始化器中创建单件，保证线程安全
    private static Singleton uniqueInstance = new Singleton();
    
    private Singleton() { };
    
    public static Singleton getInstance() {
        return uniqueInstance;
    }
}
```

- 用“双重检查锁”，在 getInstance() 中减少同步。利用双重检查锁，首先检查实例是否已经创建，如果尚未创建，才进行同步，这样只有第一次才需要同步操作，


```java
public class Singleton {
    // 关键字 volatile 是轻量级的同步机制，确保当变量 uniqueInstance 被初始化
    // 成 Singleton 实例时，多个线程正确处理 uniqueInstance 变量
    private volatile static Singleton uniqueInstance;
    
    private Singleton() { }
    
    public static Singleton getInstance() {
        // 检查实例，不存在则进入同步区域
        if (uniqueInstance == null) {
            // 只有第一次才执行同步操作
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

### 总结

- 单件模式确保程序中一个类只有一个实例；
- 单价模式也提供访问这个实例的全局点；
- 在 Java 模式中实现单件模式，需要私有构造器、一个静态方法和一个静态变量；
- 如果使用多个类类加载器，可能导致单件失效而产生多个实例。每个类加载器 ClassLoader 都定义一个命名空间，如果有两个以上的类加载器，不同的类加载器可能会加载同一个类，从整个程序来看，同一个类被加载多次。发生在单件上，就会产生多个单件并存的怪异现象。解决方法是自行指定类加载器，并制定同一个类加载器。
