
## 装饰者模式（Decorator Pattern）
　　装饰者模式动态地将责任附加到对象上（在运行时动态添加），若要扩展功能，装饰者提供了比继承更有弹性的替代方法，不需要修改原有的代码。<br />
　　如果依赖继承，那么类的行为只有在编译时静态决定。换句话说，行为如果不是来自超类，就是子类覆盖后的版本，每当需要新行为时，需要修改现有的代码，并重新编译。反之，利用组合，可以在运行时实现新的装饰者增加新的行为。

### 问题
　　有一间咖啡店准备更新自己的订单系统，下面是它的类设计，顾客购买咖啡时，可以要求在其中加入各种调料，比如豆浆、摩卡等，根据加入的调料收入不同的费用，订单系统需考虑到这些调料部分。

- Beverage（饮料）是一个抽象类，店内所提供的饮料都必须继承自此类；
- description（叙述）的实例变量，由每个子类设置，用来描述饮料，例如“超优深焙（Dark Roast）咖啡豆”，利用 getDescription() 方法返回此叙述；
- cost() 方法是抽象的，子类必须定义自己的实现。

![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_3/chapter_3_p1.png)

　　行不通的方法：

- **子类继承超类。**每当有一种新品种咖啡，则继承超类，这样会导致类爆炸，并且没考虑到如果某种调料（牛奶）价格上涨、新增焦糖调料风味等情况，造成维护困难；
- 在超类中加入各种饮料方法的实现。
    1. 调料价钱的改变会使我们更改现有的代码；
    2. 一旦出现新的调料，就需要加上新的方法，并改变超类中的 cost() 方法；
    3. 以后可能会开发出新饮料，对这些饮料而言（比如，冰茶），某些饮料可能并不适合，但是在这个设计方式中，（Tea）茶子类仍将继承那些不适合的方法，例如：hasWhip()（加奶泡）；
    4. 如果有顾客想要双倍的摩卡咖啡，又该怎么办？
    
　　**设计原则，类应该对扩展开放，对修改关闭。**比如观察者模式，通过加入新的观察者，可以在任何时候扩展 Subject（主体），而且不需要向主体中添加代码。装饰者也是遵循开放-关闭原则。

### 认识装饰者模式
　　继承无法解决问题，相反会带来类数量爆炸、设计死板，以及基类加入的新功能并不使用于所有的子类。<br />
　　新的做法，以饮料为主体，然后在运行时以调料来“装饰”饮料，比如说，顾客想要摩卡和奶泡深焙咖啡：
  
- 拿一个深焙咖啡（DarkRoast）对象；
- 以摩卡对象装饰它；
- 以奶泡对象装饰它；
- 调用 cost() 方法，并依赖委托（delegate）将调料的价钱加上去；

### 以装饰者构造饮料订单

- 以 DarkRoast 对象开始，DarkRoast 对象继承自 Beverage，且有一个用来计算饮料价钱的 cost() 方法；
- 顾客想要摩卡，则建立一个 Mocha 对象，并用它将 DarkRoast 对象包装（wrap）起来。Mocha 对象是一个装饰者，它的类型“反映了”它所装饰的对象（Beverage）。“反映”，指的是两者类型一致。所以 Mocha 对象也有一个 cost() 方法。通过多态，也可以把 Mocha 所包裹的任何 Beverage 当做是 Beverage（因为 Mocha 是 Beverage 的子类型），这就需要定义一个接口，装饰者和被装饰者都实现相同的接口；
- 顾客也想要奶泡，所以建立一个 Whip 装饰者，并用它将 Mocha 对象包起来。DarkRoast 继承自 Beverage，且有一个 cost() 方法，用来计算饮料价格。Whip 是一个装饰者，所以它也反映了 DarkRoast 类型，并包括一个 cost() 方法。所以，被 Mocha 和 Whip 包起来的 DarkRoast 对象仍然是一个 Beverage 对象，仍然可以具有 DarkRoast 的一切行为，包括调用它的 cost() 方法；
- 计算顾客价钱，通过调用最外圈装饰者（Whip）的 cost() 就可以办得到。Whip 的 cost() 会先委托它装饰的对象（也就是 Mocha）计算出价钱，然后再加上奶泡的价钱。

![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_3/chapter_3_p2.png)

　　装饰调用过程如下：

- 首先，调用最外圈装饰者 Whip 的 cost()；
- Whip 调用 Mocha 的 cost();
- Mocha 调用 DarkRoast 的 cost();
- DrakRoast 返回它的价钱 0.99；
- Mocha 在 DarkRoast 的结果上，加上自己的价钱 0.2，返回新的价钱 1.19；
- Whip 在 Mocha 的结果上，加上自己的价钱 0.1，返回新的价钱 1.19。

### 代码实现

- **抽象组件，抽象基类，用于继承，继承的目的是保持超类和子类都同属于类型，达到类型匹配，方便多个子类进行组合嵌套；**
```java
public abstract class Beverage {
    String description = "Beverage";

    public String getDescription() {
        return description;
    }

    public abstract double cost();
}
```

- **继承超类的具体组件；**

```java
public class Espresso extends Beverage {

    /**
     * 设置饮料的描述，description 实例变量继承自 Beverage
     */
    public Espresso() {
        description = "Espresso";
    }

    public double cost() {
        return 1.99;
    }
}

public class HouseBlend extends Beverage {

    public HouseBlend() {
        description = "House Blend Coffee";
    }

    public double cost() {
        return 0.89;
    }
}
```

-  **抽象装饰者，用于继承，需让 CondimentDecorator 能够取代 Beverage，所以将 CondimentDecorator 扩展自 Beverage 类；**

```java
public abstract class CondimentDecorator extends Beverage {

    /**
     * 所有调料装饰者必须重新实现 getDescription() 方法
     * @return
     */
    public abstract String getDescription();
}
```

- **具体装饰者，扩展自 CondimentDecorator；**

```java
public class Mocha extends CondimentDecorator {

    Beverage beverage;

    /**
     * 使用变量保存引用 Beverage，这样 getDescription() 和 cost()
     * 方法，可直接使用该变量的引用获得描述和超类价格，可组合不同
     * 调用的，只要构造器中引用了 Beverage
     *
     * 把饮料当做构造器的参数，再由构造器将此饮料保存在实例变量中。
     * 即用一个实例变量记录被装饰者，使得 Mocha 可以引用 Beverage
     * @param beverage
     */
    public Mocha(Beverage beverage) {
        this.beverage = beverage;
    }

    /**
     * 饮料加调料的描述，利用委托得到饮料的描述，然后在其后
     * 加上附加的叙述（如 "Mocha" )
     * @return
     */
    public String getDescription() {
        return beverage.getDescription() + ", Mocha";
    }

    /**
     * 计算带 Mocha 饮料的价钱，调用被装饰的对象 beverage.cost，加上
     * Mocha 的价钱，得到最后的结果
     * @return
     */
    public double cost() {
        return 0.2 + beverage.cost();
    }
}


public class Soy extends CondimentDecorator {

    Beverage beverage;

    public Soy(Beverage beverage) {
        this.beverage = beverage;
    }

    public String getDescription() {
        return beverage.getDescription() + ", Soy";
    }

    public double cost() {
        return 0.2 + beverage.cost();
    }
}
```

- **测试代码，下面就体现了继承的好处，因为都属于同一对象类型，所以可以进行组合嵌套，从而达到装饰的目的，即在相同对象上随意组合以添加新功能。**

```java
public class StarbuzzCoffee {

    public static void main(String args[]) {
        Beverage beverage = new Espresso();
        System.out.println(beverage.getDescription() + " $ " + beverage.cost());

        Beverage beverage2 = new HouseBlend();
        beverage2 = new Soy(beverage2);
        beverage2 = new Mocha(beverage2);

        // 同样写法
        Beverage beverage3 = new HouseBlend();
        beverage3 = new Mocha(new Soy(beverage3));
        System.out.println(beverage2.getDescription()  + beverage2.cost());
        System.out.println(beverage3.getDescription()  + beverage3.cost());
    }
}
```

### 现实场景的装饰者，Java I/O
　　I/O 的许多类都是装饰者，下面是一个典型的对象集合，用装饰者来将功能结合起来，以读取文件数据：
  
- FileInputSream，被装饰的“组件”，Java I/O 提供了几个组件，包括了 FileInputStream、SringInputSream、ByteInputSream ... 等，这些类都提供了最基本的字节读取功能；
- BufferedInputStream，具体的装饰者，加入两种行行为，利用缓冲输入来改进新能，用一个 readline() 方法（用来一次读取一行文本输入数据）来增强接口；
- LineNumberInputStream，具体的装饰者，加上计算行数的功能；

　　BufferedInputSream 及 LineNumberInputStream 都扩展自 FilterInputStream，而 FilterInputStream 是一个抽象的装饰类。
  
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_3/chapter_3_p3.png)

### 装饰者模式的缺点

- 由于会在设计中加入大量的小类，导致别人不容易理解其设计方法，比如 Java I/O 。但是如果能认识到这些类都是用来包装 InputStream 的，就简单多了；
- 类型问题。有些时候，人们在客户代码中依赖某种特殊类型，然后忽然导入装饰者，却又没有周祥地考虑一切。有些代码会依赖特定的类型，而这样的代码一导入装饰者，就会出问题。因为装饰者为了保证类型相同，都是基础自超类的；
- 采用装饰者在实例化组件时，会增加代码的复杂度。一旦使用装饰者模式，不只需要实例化组件，还要把此组件包装进装饰者中，可能会有多个组件。

### 总结

- 在设计中，应该允许行为可以被扩展，而无需修改现有的代码；
- 组合和委托可用于在运行时动态加上新的行为；
- 除了继承，装饰者模式也可以让我们扩展行为；
- 装饰者和被装饰者对象有相同的超类型。装饰者类反映出被装饰的组件类型，它们具有相同的类型，都经过接口或继承实现。既然装饰者和被装饰者对象有相同的超类型，所以在任何需要原始对象（被包装的）场合，可以用装饰过的对象代替它；
- 可以用一个或多个装饰者包装一个对象。装饰者可以在被装饰者的行为前面与/ 或后面加上自己的行为，甚至将被装饰者的行为整个取代掉，达到特定的目的；
-  对象可以在任何时候被装饰，所以可以在运行时动态地、不限量地用你喜欢的装饰者来装饰对象。
- 通过继承使得装饰者和被装饰者是一样的类型，也就是有共同的超类。**即利用继承达到“类型匹配”，而不是利用继承获得“行为”；**
- 装饰者和被装饰者有相同的“接口”，是为了装饰者能代替被装饰者；
- 将装饰者与组件组合时，就是在加入新的行为，所得到的新行为，并不是继承自超类，而是由组合对象得来的；

　　总结下，继承 Beverage 抽象类，是为了有正确的类型，而不是继承它的行为。行为来自装饰者和基础组件，或与其它装饰者之间的组合关系。使用对象组合，可以把所有饮料和调料更有弹性地加以混合与匹配。
