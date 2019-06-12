
## 简单工厂模式（Simple Factory Pattern）
　　除了使用 new 操作符外，还有其它制造对象的方法，工厂模式就是其中一种。工厂方法模式定义了一个创建对象的接口，子类通过继承超类来决定要实例化的类是哪一个。<br />
　　工厂方法让类把实例化推迟到子类。工厂方法针对抽象编程，而不针对具体类编程。

### new 的缺点
　　当使用 new 时，是在实例化一个具体类。而代码绑着具体类会导致代码更脆弱，缺乏弹性。比如有一群相关的具体类时：
  
```java
Duck duck;

if (picnic) {
    duck = new MallardDuck();
} else if (hunting) {
    duck = new DecoyDuck();
} else if (inBathTub) {
    duck = new RubberDuck();
}
```

　　要实例化哪个类，要在运行时才知道。一旦有变化和扩展，就必须重新打开这段代码进行检查和修改，通常这样修改过的代码将造成部分系统更难维护和更新，而且更容易犯错。<br />
　　**针对接口编程，可以隔离掉以后系统可能发生的一大堆改变，即通过多态，可以与任何新类实现该接口。** 而如果使用大量的具体类时，一旦加入新的具体类，就必须改变代码，并非“对修改关闭”。在 Spring 里，也是将 new 的实现交由第三方 Spring 来控制，避免耦合。<br />
　　解决方法，根据设计原则，找出会变化的部分，把它们从不变的部分分离出来。即将实例化具体类的代码从应用中抽离或封装起来，使它们不会干扰应用的其它部分。<br />
　　如下，是使用大量具体类的代码，一旦要改变，则需要修改代码：
  
```java
public  Pizza orderPizza(String type) {
    Pizza pizza;
    
    if (type.equals("cheese")) {
        pizza = new CheesePizza();
    } else if (type.equals("clam")) {
        pizza = new ClamPizza();
    } else if (type.equals("veggie") {
        pizza = new VeggiePizza();
    }
    
    // 不变的部分
    pizza.prepare();
    pizza.bake();
    pizza.cut();
    pizza.box();
    return pizza;
}
```

　　根据设计原则，将变化的部分提取出来，即创建 pizza 的部分，而 pizza 的处理则是不变的部分。

### 封装创建堆对象的代码
　　将创建对象移到 orderPizza() 之外，即移到另一个对象中，由这个对象专职创建 pizza：
  
```java
/**
 * SimplePizzaFactory 用于创建披萨，这个工厂内定义一个 createPizza() 方法，
 * 所有客户用这个方法来实例化对象。该工厂 SimplePizzaFactory 可以服务多个
 * 类，不局限于 orderPizza。当以后要改变时，只需修改这个类即可，其它是固定
 * 不变的。
 */
public class SimplePizzaFactory {

    public Pizza createPizza(String type) {
        Pizza pizza = null;

        if (type.equals("cheese")) {
            pizza = new CheesePizza();
        } else if (type.equals("clam")) {
            pizza = new ClamPizza();
        } else if (type.equals("veggie") {
            pizza = new VeggiePizza();
        }
        return pizza;
    }
}
```

### 重做 PizzaStore 类
```java
public class PizzaStore {
    SimplePizzaFactory factory;
    
    // 构造器，传入一个工厂参数，可根据需求改变工厂，创建不同披萨
    public PizzaStore(SimplePizzaFactory factory) {
        this.factory = factory;
    }
    
    // 不变的部分，不需要改动代码
    public Pizza orderPizza(String type) {
        Pizza pizza;
        
        // 把 new 操作符替换成工厂对象的创建方法，不再使用具体实例化。
        // 即将创建对象的方法移到另一个类，通过构造器的参数来传入
        pizza = factory.createPizza(type);
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
}
```

### 定义简单工厂
　　简单工厂不是一个设计模式，像是一种编程习惯。

### 加盟披萨店
　　使用 SimplePizzaFactory，写出三种不同工厂，分别是 NYPizzaFactory、ChicagoPizzaFactory、CalifornialPizzaFactory，如下：
  
```java
// 创建工厂，是制造纽约风味的披萨
NYPizzaFactory nyFactory = new NYPizzaFactory();
// 建立披萨店，将纽约工厂作为参数
PizzaStorel nyStore = new PizzaStore(nyFactory);
// 制造披萨
nyStore.orderPizza("Veggie");

ChicagoPizzaFactory chicagoFactory = new ChicagoPizzaFactory();
PizzaStorel chicagoStore = new PizzaStore(chicagoFactory);
chicagoStore.orderPizza("Veggie");
```

### 允许子类做决定
　　超类定义一个抽象方法，各个子类通过继承超类 PizzaStore 实现抽象方法 createPizza，让每个子类各自决定如何制造披萨。<br />
　　如下代码中，每个子类实现各自的 createPizza，而 OrderPizza 是不变的。
  
```java
public abstract class PizzaStore {
    public Pizza OrderPizza(String type) {
        Pizza pizza;
        
        pizza = createPizza(type);
        
        pizza = factory.createPizza(type);
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
    
    // 工厂方法是抽象的，需要实现
    abstract Pizza createPizza(String type);
}
```

　　继承超类，实现子类的抽象方法。原本由一个对象负责所有具体类的实例化，通过继承 PizzaStore，变成由一群子类来负责实例化。注意，超类的 OrderPizza() 方法，并不知道在创建的披萨是哪一种，只知道这个披萨可以被准备、被烘烤、被切片和被装盒。

```java
public class NYPizzaStore extends PizzaStore {
    Pizza createPizza(String item) {
        if (item.equals("cheese")) {
            return new NYStyleCheesePizza();
        } else if (item.equals("veggie")) {
            return new NYStyleCheesePizza();
        } else if (item.equals("clam") {
           return new NYStyleCheesePizza();
        }
        return null;
    }
}
```

　　工厂方法用来处理对象的创建，并将这样的行为封装在子类中。这样，客户程序中关于超类的代码就和子类对象创建代码解耦。
  
```java
abstract Product factoryMethod(String type)
```

- abstract，工厂方法是抽象的，所以依赖子类来处理对象的创建；
- Product，工厂方法必须返回一个产品，超类中定义的方法，通常使用到工厂方法的返回值；
- factoryMethod，工厂方法将客户（也就是超类中的代码，如 orderPizza()）和实际创建具体产品的代码分隔开来。

### 总结
　　**所有工厂模式都用来封装对象的创建。** 工厂方法模式通过让子类决定该创建的对象是什么，来达到将对象创建的过程封装的目的。

　　创建者（Creator）类：
  
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_4/chapter_4_p1.png)

　　产品类：
  
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_4/chapter_4_p2.png)

　　将一个 orderPizza() 方法和一个工厂方法 createPizza() 联合起来，就可以成为一个框架。除此之外，工厂方法将生产知识封装进各个创建者，这样的做法，也可以被视为是一个框架。<br />
　　工厂方法模式能够封装具体类型的实例化。抽象的 Creator 提供了一个创建对象的方法的接口，也称为“工厂”方法。在抽象的 Creator 中，任何其它实现的方法，都可能使用到这个工厂方法制造出来的产品，但只有子类真正实现这个工厂方法并创建产品。<br />
　　工厂方法让子类决定要实例化的类是哪一个，即在编写创建者类时，不需要知道实际创建的产品是哪一个。选择了使用哪个子类，自然就决定了实际创建的产品是什么。
  
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_4/chapter_4_p3.png)

### 依赖倒置原则
　　要依赖抽象，不要依赖具体类。不能让高层组件依赖低层组件，而且，不管高层或低层组件，两者都依赖于抽象。<br />
　　所谓高层组件，是由其它低层组件定义其行为的类。如下，PizzaStore 是个高层组件，它的行为由披萨定义的，PizzaStore 创建所有不同的披萨对象，依赖这些具体的披萨类，而披萨本身属于低层组件的。
  
```java
public class DependentPizzaStore {
    public Pizza createPizza(String style, String type) {
        Pizza pizza = null;
        
        if (style.equals("NY")) {
            if (type.equals("cheese") {
                pizza = new NYStyleCheesePizza();
            } else if (type.equals("veggie") {
                pizza = new NYStyleVeggiePizza();
            }
        } else if {style.equals("Chicago")) {
            if (type.equals("cheese") {
                pizza = new ChicagoStyleCheesePizza();
            } else if (type.equals("veggie") {
                pizza = new ChicagoStyleVeggiePizza();
            }
        } else {
            System.out.println("Error"};
            return null;            return null;

        }
        
        pizza = factory.createPizza(type);
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
}
```

　　该代码的结构如下：
  
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_4/chapter_4_p4.png)

　　根据依赖倒置原则，对于高层组件和低层组件，要依赖抽象类，而不依赖具体类，使用工厂方法，将实例化对象的代码独立出来，如下高层组件（PizzaStore）和低层组件（披萨类）都依赖于 Pizza 抽象：

![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_4/chapter_4_p5.png)

　　尽量遵循以下几点可避免违反依赖倒置原则：
  
- **变量不可以持有具体类的引用。** 如果使用 new，就会持有具体类的引用，可以用工厂来避开这样的做法；
- **不要让类派生自具体类。** 如果派生自具体类，就会依赖具体类，请派生一个抽象（接口或抽象类）；
- **不要覆盖基类中已实现的方法。** 如果覆盖基类已实现的方法，难么你的基类就不是一个真正适合继承的抽象，基类中已实现的方法，应该由所有的子类共享。

### 代码实现

- 创建抽象工厂，用于创建披萨原料家族，因为不同工厂店使用的原料不同；

```java
/**
 * 工厂方法，定义一个抽象类或接口。如果有一个通用的方法，可使用抽象类来实现，
 * 即包括通用的方法和抽象的方法，继承抽象类来实现抽象方法即可。通用方法则用于
 * 子类直接调用
 */
public interface PizzaIngredientFactory {
    // 为每个原料定义一个工厂方法
    public Dough createDough();
    public Sauce createSauce();
}

public interface Dough {
    public String toString();
}

public interface Sauce {
    public String toString();
}
```

- 通过抽象工厂所提供的接口，子类实现该接口的工厂方法，该工厂方法定义具体的类实例的实现，比如 createDough() 为创建 ThickCrustDough()，这里使用组合方法，方便其它类也可以调用 ThickCrustDough()，减少重复代码；


```java
public class NYPizzaIngredientFactory implements PizzaIngredientFactory {

    @Override
    public Dough createDough() {
        return new ThinCrustDough();
    }

    @Override
    public Sauce createSauce() {
        return new MarinaraSauce();
    }
}
```

- 具体类的实现，继承抽象接口，保持类型匹配，将原料写成多个类，方便不同子类进行组合调用；

```java
public class ThickCrustDough implements Dough {
    public String toString() {
        return "Thick Crust Dough";
    }
}

public class MarinaraSauce implements Sauce {
    public String toString() {
        return "MarinaraSauce";
    }
}
```

- 低层组件，具体类实现抽象类；

```java
public class CheesePizza extends Pizza {

    PizzaIngredientFactory ingredientFactory;
    
    // 构造器传入工厂参数
    public CheesePizza(PizzaIngredientFactory ingredientFactory) {
        this.ingredientFactory = ingredientFactory;
    }
    
    // 实现抽象方法
    void prepare() {
        System.out.println("Preparing " + name);
        // 调用工厂的方法
        dough = ingredientFactory.createDough();
        sauce = ingredientFactory.createSauce();
    }
}

public class ClamPizza extends Pizza {

    PizzaIngredientFactory ingredientFactory;

    public ClamPizza(PizzaIngredientFactory ingredientFactory) {
        this.ingredientFactory = ingredientFactory;
    }

    void prepare() {
        System.out.println("Preparing " + name);
        dough = ingredientFactory.createDough();
        sauce = ingredientFactory.createSauce();
    }
}
```

- 抽象类 PizzaStore，具体类 NYPizzaStore，通过获取工厂实例，作为参数传入不同的 pizza 子类，来实现创建 pizza 类。

```java
public abstract class PizzaStore {
    
    //  抽象工厂方法，不同子类的实现不同
    protected abstract Pizza createPizza(String item);
    
    // 通用方法
    public Pizza orderPizza(String type) {
        Pizza pizza = createPizza(type);
        System.out.println("--- Making a " + pizza.getName() + " ---");
        pizza.prepare();
        pizza.bake();
        return pizza;
    }
}

public class NYPizzaStore extends PizzaStore {
    Pizza pizza = null;
    // 获取工厂实例
    PizzaIngredientFactory pizzaIngredientFactory = new NYPizzaIngredientFactory();

    @Override
    protected Pizza createPizza(String item) {
        if (item.equals("cheese")) {
            // 通过工厂，创建不同的 pizza 子类
            pizza = new CheesePizza(pizzaIngredientFactory);
            pizza.setName("New York style cheese pizza");
        } else if (item.equals("clam")) {
            pizza = new ClamPizza(pizzaIngredientFactory);
            pizza.setName("New York style clam pizza");
        }
        return pizza;
    }
}
```

　　总结：
    
- 引入新类型的工厂，即抽象工厂，来创建披萨原料家族；
- 通过抽象工厂所提供的接口，创建产品的家族，利用这个接口书写代码，这样就从实际工厂解耦，以便在不同上下文中实现各式各样的工厂，制造出各种不同的产品。比如：不同呢的区域、不同的操作系统、不同的外观及操作；
- 因为代码从实际的产品中解耦，所以可以替换不同的工厂来取得不同的行为。比如替换不同的工厂，获取不同的原料

### 定义抽象工厂模式
　　抽象工厂模式提供一个接口，用于创建相关或依赖对象的家族，而不需要明确指定具体类。<br />
　　抽象工厂允许客户使用抽象的接口来创建一组相关的产品，而不需要知道（或关心）实际产出的具体产品是什么。这样一来，客户就从具体的产品中被解耦。
  
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_4/chapter_4_p6.png)

　　抽象工厂的方法经常以工厂方法的方式实现，比如 createDough()、createSource() 等，每个方法都被声明成抽象，而子类的方法覆盖这些方法来创建某些对象。抽象工厂的任务是定义一个负责创建一组产品的接口，这个接口内的每个方法都负责创建一个具体产品，同时利用实现抽象工厂的子类来提供这些具体的做法。

### 工厂方法与抽象工厂的区别

- 抽象工厂与工厂方法都是负责创建对象的，工厂方法使用继承类来创建对象，而抽象工厂通过对象组合方式；
- 利用工厂方法创建对象，需要扩展一个类，并覆盖它的工厂方法来创建对象。其实整个工厂方法模式，**就是通过子类来创建对象。** 通过这种方法，客户只需要知道他们所使用的抽象类型就可以了，而由子类来负责决定具体类型，即工厂方法负责将客户从具体类型中解耦，由子类来实现具体类型，客户知道抽象类型就行；
- 抽象工厂提供一个用来创建一个产品家族的抽象类型，即这个类型的子类定义了产品被产生的方法。要想使用整个工厂，必须先实例化它，然后将它传入一些针对抽象类型所写的代码中，即把一群相关的产品集合起来，同样是把客户从所使用的实际具体产品中解耦。
- 工厂方法，在一个类中提供一个抽象接口，然后子类实现该抽象接口即可；

![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_4/chapter_4_p7.png)

### 总结

- 所有的工厂都是用来封装对象的创建；
- 简单工厂，虽然不是真正的设计模式，但仍不失为一个简单的方法，可以将客户程序从具体类解耦；
- 工厂方法使用继承，把对象的创建委托给子类，子类实现工厂方法来创建对象；
- 抽象工厂使用对象组合，对象的创建被实现在工厂接口所暴露出来的方法中；
- 所有工厂模式都通过减少应用程序和具体类之间的依赖促进松耦合，即使用抽象类来依赖，高层组件和低层组件都依赖抽象类；
- 工厂方法允许类将实例化延迟到子类进行，即继承超类，实现抽象方法；
- 抽象工厂创建相关的对象家族，而不需依赖它们的具体类；
- 依赖倒置原作，指导我们避免依赖具体类型，而要尽量依赖抽象；
- 工厂是很有威力的技巧，帮助我们针对抽象编程，而不要针对具体类编程。
