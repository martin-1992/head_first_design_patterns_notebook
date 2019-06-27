## head first 设计模式笔记

### [策略模式（Strategy Pattern）](https://github.com/martin-1992/head_first_design_patterns_notebook/tree/master/chapter_1)
　　会变化的部分抽取出来实现为行为类，通过 new 来调用，不会变化的部分则为超类，由子类继承。
  
- 将会变化的部分抽取出来，声明为（行为）接口类型，比如飞行接口。由子类实现不同的具体行为动作类，比如继承飞行接口实现的不会飞的动作、会飞的动作；
- 不会变化的部分实现为超类中的通用方法，比如有一个超类为鸭子类，通用方法为游泳，对象子类继承超类；
- 鸭子子类根据各自要求调用具体的行为动作类。一种鸭子子类引用不会飞的动作类，另一种鸭子子类引用会飞的动作类。
　　
  
　　应用场景：Runnable 的 run 方法和 Thread 类本身的 run 方法，都是将线程的控制本身和业务逻辑的运行分离开来。 Runnable 接口的 run 方法，即业务逻辑，由我们自己实现，其他不会变化的部分，比如启动、结束线程这些，则由超类定义好。另外还用到了模板方法，启动流程的步骤都定义好，只要实现其中一部分细节，业务逻辑就可以。<br />
　　无论是 Runnable 的 run 方法，还是 Thread 类本身的 run 方法（事实上 Thread 类也是实现 Runnable 接口）都是想将线程的控制本身和业务逻辑的运行分离开来。

### [观察者模式（Observer Pattern）](https://github.com/martin-1992/head_first_design_patterns_notebook/tree/master/chapter_2)
　　当有最新消息时通知对象，JavaBeans、RMI、MVC 中都有用到观察者模式。

- 面向接口编程，定义主题和观察者接口；
- 主题接口，实现该接口，即为主题，可通知观察者，一般包含三个方法，注册、删除、通知观察者；
- 观察者接口，所有观察者必须实现该接口的 update() 方法。因为主题在通知所有观察者时，会调用每个观察者的 update 方法；
- 实现主题接口，实即为主题对象。首先创建一个数组，注册、删除观察者即是在数组中添加或删除观察者对象，通知观察者也是通过数组遍历，调用每个观察者的 update() 方法；
- 实现观察者接口，即为观察者对象，可收到主题推送的信息，通过构造器将主题作为参数传入，然后调用主题方法注册观察者。

### [装饰者模式（Decorator Pattern）](https://github.com/martin-1992/head_first_design_patterns_notebook/blob/master/chapter_3/README.md)
　　用于在原有代码功能上加上新的功能，不需要修改现有代码，动态添加。比如 BufferedInputStream 在 InputStream 基础上提供了缓冲区功能。

- 定义一个超类为抽象组件，所有对象类都继承该类，继承的目的是保证超类和子类都同属于类型，达到类型匹配，方便多个子类进行组合嵌套，定义抽象方法，比如价格；
- 继承超类的组件，为抽象装饰者，比如咖啡类，实现抽象方法，价格为 15；
- 继承超类的具体装饰者，构造器中需传入抽象装饰者，因为都都继承同一类，调用相同的方法，比如摩卡的价格 + 构造器传入的继承超类组件的价格，以此类推，奶泡价格 + 摩卡价格 + 咖啡价格，new Whip(new Mocha(new Caffe))；

### [简单工厂模式（Simple Factory Pattern）](https://github.com/martin-1992/head_first_design_patterns_notebook/blob/master/chapter_4/README.md)
　　所有工厂模式都是用来封装对象的创建。工厂方法模式通过让子类决定该创建的对象是什么，来达到将对象创建的过程封装的目的。

- 超类定义一个抽象方法，为创建对象的部分；
- 子类继承超类，实现自己的创建对象，即通过子类来创建对象。

### 抽象工厂模式（Abstract Factory Pattern)

### [单件模式（Singleton Pattern）](https://github.com/martin-1992/head_first_design_patterns_notebook/blob/master/chapter_5/README.md)
　　确保类只有一个实例，常用于管理共享资源（数据库连接池、线程池）、缓存、对话框、处理偏好设置和注册表的对象、日志对象，这类对象只能有一个实例，多个实例会出问题。

- 利用静态变量（比如 uniqueInstance）来记录该类的唯一实例；
- 调用 getInstance() 创建实例，考虑到多线程的情况下，使用双重检查锁来判定实例是否创建，如没则创建实例，并赋值给 uniqueInstance 变量保存。如已创建，这直接放回 uniqueInstance 变量。

### [命令模式（Command Pattern）](https://github.com/martin-1992/head_first_design_patterns_notebook/blob/master/chapter_6/README.md)
　　将对象的方法封装在一个命令对象的接口上，进行调用。命令模式用在队列请求中，队列不需要知道对象是谁，只要调用 execute() 即可。
  
- 创建一个命令接口，包含 execute() 和 undo() 方法；
- 所有命令对象需实现命令接口，将自身方法封装在 execute() 中，比如电灯对象的打开电灯方法封装在 execute() 中，冰箱对象的打开冰箱方法封装在 execute() 中。用户不需要知道对象的方法是怎么运作的，只需要调用 execute() 方法即可；
- 宏命令，是将一组命令对象放在数组中，然后遍历调用 execute() 方法。

### [适配器模式（（Command Pattern）](https://github.com/martin-1992/head_first_design_patterns_notebook/blob/master/chapter_7/README.md)
　　有两个接口对象，通过适配器将一个接口对象转换成另一个期望的接口，本质上是封装不兼容对象的方法，即通过实现期望的接口，传入要被转换的对象，将被转换对象的方法封装一层期望接口的方法，这样通过期望接口的方法调用的是被转换对象的方法。如下，
  
- 有两个接口对象，枚举接口 Enumerattion 和 迭代接口 Iterator；
- Enumerattion 接口有两个方法分别为 hasMoreElements() 和 nextElements()，Iterator 接口有三个方法分别为 hasNext()、next()、remove() 方法；
- 适配器将不兼容的 Enumerattion 接口包装起来，兼容 Iterator 对象；
- 构建一个适配器对象，实现 Iterator 接口，然后传入 Enumerattion 对象；
- 实现 Iterator 接口的方法，hasNext() 里封装调用 Enumerattion 对象的 hasMoreElements() 方法，next() 里封装调用 Enumerattion 对象的 nextElements()，由于 Enumerattion 没有对应的 Iterator 接口的 remove() 方法，则抛出异常；

### [外观模式（Facade Pattern）](https://github.com/martin-1992/head_first_design_patterns_notebook/blob/master/chapter_7/README.md)
　　提供了一个统一的接口，用来访问子系统中的一群接口，它定义了一个高层接口，让子系统更容易使用。
  
- 新建一个类；
- 将需要多个类的调用方法封装在一个类的方法中，比如将多个动作封装在一个动作方法中，进行调用。

### [模板方法模式（Template Method Pattern）](https://github.com/martin-1992/head_first_design_patterns_notebook/blob/master/chapter_8/README.md)
　　模板方法模式在一个方法中定义一个算法的骨架，将一些步骤通过定义抽象方法延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤，即子类决定如何实现模板算法中的某些步骤。以 Java 中的排序方法为例，抽象方法为 compareTo，交由子类实现，如何比较两个对象大小，其余由超类的算法步骤定义好。

- 抽象类定义一个算法，包含多个方法步骤的调用；
- 这些方法步骤中，不会变的部分作为超类中的通用方法，会变的部分作为超类中的抽象方法，交由子类实现；

　　**应用场景：**
- Thread 的 run 和 start 是一个典型的模板设计模式，父类编写算法结构代码，子类实现逻辑细节 run，即线程的启动步骤都由父类实现好了，按照顺序启动，只需实现步骤 run 方法的细节即可；
- Netty 中的抽象类 SimpleInBoundHandler，在使用 channelRead 方法进行读数据中，只需实现抽象方法 channelRead0，而不需管处理消息后的内存释放，这在模板方法中已经定义好了；
- Tomcat 中的 [LifecycleBase](https://github.com/martin-1992/Tomcat-Notes/blob/master/LifeCycle/LifecycleBase.md) 类，为抽象基类，定义了关于生命周期的方法如 init()、start()、stop()、destroy() 等。以 init() 为例，该方法的流程是先进行生命周期状态转换，并检查是否有监听器监听该状态的事件，有则调用监听器。这些是在方法中定义的，子类只要继承该类，实现抽象方法 initInternal() 即可。


### 迭代器模式（Iterator Pattern）
　　将集合框架的使用方法使用一个通用接口封装，通过该接口的 hasNext()、next() 和 remove() 方法进行调用。

- 定义迭代器接口，方法有 hasNext()、next() 和 remove()；
- 实现迭代器接口，为具体迭代器，构造器传入集合框架，使用迭代器的方法封装传入集合框架的方法，比如迭代器接口 next() 方法封装数组的下标的使用等。

### 组合模式（Composite Pattern）


### [状态模式（State Pattern）](https://github.com/martin-1992/head_first_design_patterns_notebook/blob/master/chapter_10/README.md)
　　状态模式是不用在 context 中放置许多条件判断的替代方案，通过将不同状态的行为封装进不同的状态对象中，使用 getter() 来获得状态对象，使用 setter() 对其进行状态转换。与策略模式区别在于，状态模式的状态类会随着时间而改变，而策略模式一般是指定了具体的行为状态类，不会改变。
  
- 定义一个接口或抽象类，如果没有通用方法，则定义为接口，有则定义为抽象类；
- 继承实现该接口或抽象类，为状态类，将每个状态的行为局部到它自己的类中，并规定了状态的转换，比如没有放硬币到糖果机的状态，只能转换到投了硬币的糖果机的状态；
- 状态间通过客户类（主体，构造器传入糖果机的引用）的 getter() 和 setter() 来进行状态转换。

### [代理模式（Proxy Pattern）](https://github.com/martin-1992/head_first_design_patterns_notebook/blob/master/chapter_11/README.md)
　　代理模式同装饰者模式一样，都是用一个对象把另一个对象包装起来，区别在于装饰者为对象增加行为，而代理模式是控制对象的访问。代理模式和适配器一样都是挡在其它对象的前面，并负责将请求转发给他们。适配器会改变对象适配的接口，而代理则实现相同的接口。代理模式还可以根据权限只提供给客户部分接口。

- 为代理和被代理对象创建一个接口，都为同一个接口，用于包装另一个对象（封装被代理对象的方法），并控制对它的访问，即拦截被代理对象的调用方法由代理来决定是否调用；
- 因为为同一个接口，所以传入被对象的类型，调用的是代理对象的方法，在由代理对象（通过反射）来调用被代理对象的方法；
- 反射可检查要调用的方法是否被允许的，从而达到使用代理来控制对象的访问。

### 一句话概括各模式

- 策略模式，将会变化的部分提取出来作为接口，调用者通过组合各接口的具体行为类来调用不同的行为；
- 观察者模式，创建一个数组，将观察者对象放入该数组，遍历该数组即通知；
- 装饰者模式，包装一个对象，加上不同的方法行为；
- 简单工厂模式，对象的创建由子类继承实现；

### reference：
https://design-patterns.readthedocs.io/zh_CN/latest/read_uml.html <br />
https://quanke.gitbooks.io/design-pattern-java/
