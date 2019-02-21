
## 适配器模式（Adapter Pattern）与外观模式（Facade Pattern）
　　适配器模式，用于包装某些对象，将一个不兼容接口的对象包装起来，变成兼容的对象，即将类的接口转换成想要的接口，以便实现不同的接口。<br />
  
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_7/chapter_7_p1.png)
 
　　外观模式，将对象包装起来以简化其接口。

### 面向对象适配器
　　适配器就像一个中间人，将新厂商的接口转换成你所期望的接口。<br />
　　举例，对鸭子类使用适配器，使其看起来像火鸡，即包装了鸭子适配器的火鸡：

- 鸭子类和火鸡类接口，及其子类实现；
  
```java
// 鸭子类实现 Duck 接口
public interface Duck() {
    public void quack();
    public void fly();
}

// 子类实现超类的接口
public class MallardDuck implements Duck {
    public void quack() {
        System.out.println("Quack");
    }
    public void fly() {
        System.out.println("flying");
    }
}

// 超类，火鸡类，面向接口编程
public interface Turkey {
    public void gobble();
    public void fly();
}

public class WildTurkey implements Turkey {
    public void gobble() {
        System.out.println("gobble");
    }
    public void fly() {
        System.out.println("flying aa");
    }
}
```

- 鸭子适配器，构造器传入火鸡类对象，使用该对象实现鸭子类的接口；
    1. 实现想要转换成的类型接口，即客户所期望看到的接口；
    2. 需要取得适配的对象 Turkey 的引用，利用构造器来取得；
    3. 实现 Duck 接口中的所有方法。
  
```java
public class TurkeyAdapter implements Duck {
    Turkey turkey;
    
    // 构造器，传入需要适配的对象
    public TurkeyAdapter(Turkey turkey) {
        this.turkey = turkey;
    }
    // 实现 Duck 接口中的方法
    public void quack();
        turkey.gobble();
    } 
    
    public void fly() {
        for (int i = 0; i < 5; i++) {
            turkey.fly();
        }
    }
}   
```

- 测试代码。

```java
public class DuckTestDrive {
    public static void main(String[] args) {
        // 创建鸭子类和火鸡类
        MallardDuck duck = new MallardDuck();
        WildTurkey turkey = new WildTurkey();
        // 将火鸡类包装进一个适配器中
        Duck turkeyAdapter = new TurkeyAdapter(turkey);
        
        // 测试方法需要传入鸭子对象
        testDuck(duck);
        testDuck(turkeyAdapter)
    }
    
    // 传入一个鸭子对象，并调用它的方法
    static void testDuck(Duck duck) {
        duck.quack();
        duck.fly();
    }
}

```

### 适配器模式解析
　　
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_7/chapter_7_p2.png)

　　使用适配器的过程：
  
- 客户依据目标接口（Duck）实现的；
- 适配器实现了目标接口（Duck），并（通过构造器）持有被适配者的实例 Turkey，即火鸡适配器实现了目标接口（Duck），并引用一个被适配者（Turkey）；  
- 客户通过目标接口调用适配器的方法对适配器发出请求，即目标接口（Duck 的 quack、fly）封装了实现被适配者（Turkey）接口的子类（WildTurkey）的方法；
- 客户接收到调用的结果，但不知道这是适配器在起转换作用，所以客户和被适配者是解耦的。

### 定义适配器模式
　　适配器模式将一个类的接口，转换成客户期望的另一个接口，通过传入被适配者对象，并使用期望接口的方法封装被适配者的方法。适配器让原本接口不兼容的类可以合作无间。<br />
　　有两种适配器，分别为对象适配器和类适配器，但类适配器需要继承两个类，而 Java 只允许单继承，所以这里就不讨论了。对象适配器模式的类图：
  
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_7/chapter_7_p3.png)

　　这个适配器模式充满良好的 OO 设计原则：

- 使用对象组合；
- 以修改的接口包装被适配者。

　　这种做法还有额外的优点，即被适配者的任何子类，都可以搭配适配器使用，比如火鸡适配器 turkeyAdapter，只要实现 Turkey 接口的任何子类都可以传入。<br />
　　注意，这个模式如何把客户和接口绑定起来，而不是和实现绑定起来。可以使用数个适配器，每一个都负责转换不同组的后台类。或者，也可以加上新的实现，只要它们遵守目标接口即可。

### 适配器的使用

　　Java 早期的集合（collection）类型（如 Vector、Stack、HashTable）都实现了一个名为 elements() 的方法，该方法会返回一个 Enumeration，如下：
  
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_7/chapter_7_p4.png)

　　最新的是使用 Iterator（迭代器）接口，如下：

![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_7/chapter_7_p5.png)

　　我们经常面对遗留代码，这些代码暴露出了枚举器接口，但我们又希望在新的代码中只使用迭代器，这时就需要构造一个适配器，实现目标接口 Iterator，被适配者是 Enumeration，类图如下：

![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_7/chapter_7_p6.png)

　　如下，为 Enumeration 的适配器，将枚举 Enumeration 适配成迭代器 Iterator，所以需要实现迭代器 Iterator 的接口，通过传入 Enumeration，将 Enumeration 的方法封装成 Iterator 接口的方法，即使用 Iterator 接口的方法，调用的是 Enumeration 的方法：
  
```java
public class EnumerationIterator implements Iterator {
    Enumeration enum;
    
    // 构造器，传入被适配者，用一个实例变量记录枚举
    public EnumerationIterator(Enumeration enum) {
        this.enum = enum;
    }
    
    // Iterator 的 hasNext() 方法是委托给 Enumeration 的 hasMoreElements() 方法
    public boolean hashNext() {
        return enum.hasMoreElements();
    }
    
    // Iterator 的 next() 方法是委托给 Enumeration 的 nextElement() 方法
    public Object next() {
        return enum.nextElement();
    }
    
    // Enumeration 没有实现 remove() 方法，所以这里抛出异常
    public void remove() {
        throw new UnsupportOperationException();
    }
}
```

### 外观模式
　　外观模式也是改变接口，但改变接口的原因是为了简化接口，因为它将一个或数个类的复杂的一切都隐藏在背后，只显露出一个干净美好的外观。<br />
  
　　举例，要看电影需要多个步骤，调用多个类的方法：
  
```java
// 打开爆米花机，开始爆米花
popper.on();
popper.pop();

// 灯光调暗到 10% 的亮度
lights.dim(10);

// 把屏幕放下
screen.down();

// 打开投影机
projector.on();

// 播放电影
dvd.play(movie);
```

　　通过外观模式，实现一个提供更合理的接口的外观了，即提供一个简化接口来调用复杂系统的多个接口方法，但有必要还是可以直接使用复杂系统的接口方法。<br />
　　外观模式的特征，提供简化接口的同时，依然将系统完整的功能暴露出来，以供需要的人使用。外观不止是简化了接口，也将客户从组件的子系统中解耦。<br />
　　外观模式和适配器模式都可以保证许多类，区别在于外观模式的目的是简化接口，而适配器模式的目的是将接口转换成不同接口，符合期望接口。

### 实现简化接口
　　使用组合让外观能够访问子系统中的所有组件，即新建一个类，提供一个接口整合子系统的各种类的方法。

```java
public class HomeTheaterFacade {
    PopcornPopper popper;
    Screen screen;
    TheaterLights lights;
    Projector projector;
    DvdPlayer dvd;
    
    public HomeTheaterFacade(PopcornPopper popper, Screen screen, TheaterLights lights, Projector projector, DvdPlayer dvd) {
        this.popper = popper;
        this.screen = screen;
        this.lights = lights;
        this.projector = projector;
    }
    
    // 调用外观模式的 watchMovie() 接口即可
    public void watchMovie(String movie) {
        popper.on();
        popper.pop();
        lights.dim(10);
        screen.down();
        projector.on();
        dvd.play(movie);
    }
}
```

### 定义外观模式
　　外观模式提供了一个统一的接口，用来访问子系统中的一群接口，它定义了一个高层接口，让子系统更容易使用。即要想使用外观模式，需要创建一个接口简化而统一的类，用来包装子系统中一个或多个复杂的类，外观模式允许我们让客户和子系统之间避免紧耦合。<br />
　　外观模式的类图：
  
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_7/chapter_7_p7.png)

### 最少知识原则
　　最少知识原则，也叫墨忒耳原则。在设计一个系统时，不管是任何对象，都要注意它所交互的类有哪些，并注意它和这些类是如何交互的。该原则希望我们在设计中，不要让太多的类耦合在一起，免得修改系统中的一部分，影响到其它部分，即减少了对象之间的以来，但也会导致更多的“包装”类被制造出来，以处理和其他组件的沟通，可能会导致复杂度和开发时间的增加，降低运行时的性能。<br />
　　如果许多类之间相互依赖，这个系统就会变成易碎的系统，维护困难且复杂。就任何对象而言，在该对象的方法内，应该调用属于以下范围的方法：
  
- 该对象本身；
- 被当做方法的参数而传递进来的对象，如 Key；
- 此方法所创建或实例化的任何对象，如 Doors；
- 对象的任何组件，如 engine.start()。

　　如下类，遵守最少知识原则：
  
```java
public class Car {
    // 类组件，能够调用它的方法
    Engine engine;
    
    public Car() { }
    
    public void start(Key key) {
        Doors doors = new Doors();
        // 被当做参数传进来的对象，它的方法可以被调用
        boolean authorized = key.turns();
        
        if (authorized) {
            // 调用对象组件的方法
            engine.start();
            // 调用同一个对象内的本地方法
            updateDashBoardDisplay();
            // 调用所创建或实例化的对象的方法
            doors.lock();
        } 
    }
    
    public void updateDashBoardDisplay() {
        // 更新显示}
}
```

### 总结

- 当需要使用一个现有的类而其接口并不符合需要时，可使用适配器模式；
- 当需要简化并统一一个很大的接口或一群复杂的接口时，使用外观模式；
- 适配器模式改变接口以符合客户的期望；
- 外观模式将客户从一个复杂的子系统中解耦；
- 实现一个适配器根据目标接口的大小与复杂度而定；
- 实现一个外观模式，需要将子系统组合进外观中，然后将工作委托给子系统执行，可以为一个子系统实现多个外观模式；
- 适配器模式有两种形式，对象适配器和类适配器，类适配器需要用到多重基础；
- 适配器模式将一个对象包装起来以改变其接口，装饰者模式将一个对象包装起来以增加新的行为和责任，而外观模式将一群对象“包装”起来以简化其接口。
