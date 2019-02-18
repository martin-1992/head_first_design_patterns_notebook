# head first 设计模式笔记


### 策略模式（Strategy Pattern）
　　会变化的部分抽取出来实现为行为类，通过 new 来调用，不会变化的部分则为超类，由子类继承。
  
- 将会变化的部分抽取出来，声明为（行为）接口类型，比如飞行接口。由子类实现不同的具体行为动作类，比如继承飞行接口实现的不会飞的动作、会飞的动作；
- 不会变化的部分实现为超类中的通用方法，比如有一个超类为鸭子类，通用方法为游泳，对象子类继承超类；
- 鸭子子类根据各自要求调用具体的行为动作类。一种鸭子子类引用不会飞的动作类，另一种鸭子子类引用会飞的动作类。

### 观察者模式（Observer Pattern）
　　当有最新消息时通知对象，JavaBeans、RMI、MVC 中都有用到观察者模式。

- 面向接口编程，定义主题和观察者接口；
- 主题接口，实现该接口，即为主题，可通知观察者，一般包含三个方法，注册、删除、通知观察者；
- 观察者接口，所有观察者必须实现该接口的 update() 方法。因为主题在通知所有观察者时，会调用每个观察者的 update 方法；
- 实现主题接口，实即为主题对象。首先创建一个数组，注册、删除观察者即是在数组中添加或删除观察者对象，通知观察者也是通过数组遍历，调用每个观察者的 update() 方法；
- 实现观察者接口，即为观察者对象，可收到主题推送的信息，通过构造器将主题作为参数传入，然后调用主题方法注册观察者。

### 装饰者模式（Decorator Pattern）
　　用于在原有代码功能上加上新的功能，不需要修改现有代码，动态添加。比如 BufferedInputStream 在 InputStream 基础上提供了缓冲区功能。

- 定义一个超类为抽象组件，所有对象类都继承该类，继承的目的是保证超类和子类都同属于类型，达到类型匹配，方便多个子类进行组合嵌套，定义抽象方法，比如价格；
- 继承超类的组件，为抽象装饰者，比如咖啡类，实现抽象方法，价格为 15；
- 继承超类的具体装饰者，构造器中需传入抽象装饰者，因为都都继承同一类，调用相同的方法，比如摩卡的价格 + 构造器传入的继承超类组件的价格，以此类推，奶泡价格 + 摩卡价格 + 咖啡价格，new Whip(new Mocha(new Caffe))；

## 简单工厂模式（Simple Factory Pattern）
　　所有工厂模式都是用来封装对象的创建。工厂方法模式通过让子类决定该创建的对象是什么，来达到将对象创建的过程封装的目的。

- 超类定义一个抽象方法，为创建对象的部分；
- 子类继承超类，实现自己的创建对象，即通过子类来创建对象。

## 抽象工厂模式（Abstract Factory Pattern)

## 单件模式（Singleton Pattern）
　　确保类只有一个实例，常用于管理共享资源（数据库连接池、线程池）、缓存、对话框、处理偏好设置和注册表的对象、日志对象，这类对象只能有一个实例，多个实例会出问题。

- 利用静态变量（比如 uniqueInstance）来记录该类的唯一实例；
- 调用 getInstance() 创建实例，考虑到多线程的情况下，使用双重检查锁来判定实例是否创建，如没则创建实例，并赋值给 uniqueInstance 变量保存。如已创建，这直接放回 uniqueInstance 变量。



### reference：
https://design-patterns.readthedocs.io/zh_CN/latest/read_uml.html
