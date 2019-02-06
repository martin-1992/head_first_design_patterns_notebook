
## 策略模式（Strategy Pattern）
　　策略模式定义了算法族（行为），分别封装起来，让它们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。

### 问题
　　原先有一个鸭子的超类，即 Duck，有两种方法。各种鸭子的子类继承该超类，如下：
  
![image.png](attachment:image.png)

　　现在需要让部分鸭子会飞，方法如下：

- **父类中添加会飞方法。**由于不是所有的子类鸭子都会飞，所以不能在鸭子超类中添加会飞的方法。当然也可以通过覆盖子类中的会飞方法，但是如果子类太多，导致代码在多个子类中重复；
- **利用接口，写一个会飞的接口。**虽然解决了部分会飞的问题，但接口不具有实现代码，且无法复用，所以重复代码会变多，并且如果有 8 个子类鸭子的会飞行为不同，需要修改 8 个地方。如果某个行为被修改了，其它子类鸭子的同样行为代码也需要修改。

### 分开变化和不会变化的部分
　　设计原则，找出应用中可能需要变化的地方，把它们独立出来，不要和那些不需要变化的代码混在一起。即把会变化的部分取出并“封装”起来，以便以后可以轻易改动或扩充此部分，让其它不需要变化的部分不会受到影响。<br />
　　变化的部分为会飞的部分和会叫的部分，即鸭子类的 fly() 和 quack() 会随着鸭子的不同而改变，为了要把这两个行为从 Duck 类中分开，将把它们从 Duck 类中取出来，建立一组新类来代表每个行为。

### 设计鸭子的行为
　　设计原则，针对接口编程，而不是针对实现编程。<br />
　　即鸭子的行为被封装在一组类中，此类专门提供某行为接口的实现，这样子类鸭子只需继承该行为类，不需要知道行为的实现细节。如果需要，还可以在运行时改变行为。<br />
　　如下图， 一个会飞的接口，两个实现该接口的飞行行为类，部分不会飞的鸭子子类只需继承 FlyNoWay 行为类即可，如果以后不会飞的行为类有其它改动，只需在该行为类修改即可，这样实际的“实现”不会被绑死在鸭子的子类中。换句话说，特定的具体行为编写在实现了 FlyBehavior 与 QuakcBehavior 的类中。
  
![image.png](attachment:image.png)

　　**这里“针对接口的编程”真正的意思是“针对超类型”编程。**其关键在于多态，利用多态，程序可以针对超类型编程，执行时会根据实际状况执行到真正的行为，不会被绑死在超类型的行为上。“针对超类型编程”这句话，可以更明确地说成“变量的声明类型应该是超类型，通常是一个抽象类或者是一个接口，如此，只要是具体实现此超类型的类所产生的对象，都可以指定给这个变量。这也意味着，声明类时不用理会以后执行时的真正对象类型。”

### 实现鸭子的行为
　　有两个接口，FlyBehavior 和 QuackBehavior，还有它们对应的类，负责实现具体的行为：
  
![image.png](attachment:image.png)

　　这样的设计，可以让飞行和呱呱叫的动作被其它的对象复用，因为这些行为已经与鸭子类无关了。而我们也可以新增一些行为，不会影响到既有的行为类，也不会影响“使用”到飞行行为的鸭子类。<br />
　　通过接口定义了不同的行为类，而子类通过继承不同的行为类，获得不同的方法。避免继承超类的方法，导致相同的方法无法定义。

### 整合鸭子的行为
　　鸭子现在会将飞行和呱呱叫的动作“委托”别人处理，而不是使用定义在 Duck 类（或子类）内的呱呱叫和飞行方法。
  
- 首先，在 Duck 超类中加入一个实例变量 FlyBehavior，声明为接口类型。每个鸭子对象都会动态地设置这些变量以在运行时引用正确的行为类型，如 FlyWithWings。

```java
/**
 * 超类，定义了抽象方法（行为），
 */
public abstract class Duck {
    // 为行为接口类型声明引用变量，所有鸭子子类（在同
    // 一个 package 中）都继承它们
    FlyBehavior flyBehavior;

    public Duck() {
    }

    /**
     * 抽象方法
     */
    public abstract void display();

    /**
     * 委托给行为接口类
     */
    public void performFly() {
        flyBehavior.fly();
    }

    /**
     * 具体行为方法，所有子类都继承，公有的
     */
    public void swim() {
        System.out.println("All ducks float, even decoys!");
    }


    public void setFlyBehavior(FlyBehavior fb) {
        flyBehavior = fb;
    }
}
```

- 根据行为类接口来实现针对不同子类的具体行为类，比如会飞的鸭子调用会飞的对象（行为类），不会飞的鸭子调用不会飞的对象（行为类）；

```java
/**
 * 所有飞行行为类必须实现的接口
 */
public interface FlyBehavior {
    public void fly();
}

/**
 * 飞行行为的具体实现类，会飞行
 */
public class FlyWithWings implements FlyBehavior {
    public void fly() {
        System.out.println("flying");
    }
}

/**
 * 飞行行为的具体实现类，不会飞行
 */
public class FlyNoWay implements FlyBehavior {
    public void fly() {
        System.out.println("not fly");
    }
}

public class FlyRocketPowered implements FlyBehavior {

    public void fly() {
        System.out.println("flying with rock");
    }
}
```

- 不同鸭子子类调用不同的行为实现类，会飞的和不会飞的；

```java
public class MallardDuck extends Duck {
    public MallardDuck() {
        // 飞行行为的具体实现类，会飞行
        flyBehavior = new FlyWithWings();
    }

    /**
     * 实现 Duck 抽象类的方法
     */
    public void display() {
        System.out.println("Mallard duck");
    }
}


public class ModelDuck extends Duck {

    public ModelDuck() {
        flyBehavior = new FlyNoWay();
    }

    public void display() {
        System.out.println("model duck");
    }
}
```

- 测试脚本，ModelDuck 通过 setter 方法，传输具体的行为类，从而可改变原有的行为方法。

```java
public class MiniDuckSimulator {

    public static void main(String[] args) {
        Duck mallard = new MallardDuck();
        mallard.performFly();
        mallard.swim();
        mallard.display();

        System.out.println("=====================");

        Duck model = new ModelDuck();
        // 第一次调用 performFly() 会被委托给 flyBehavior 对象（也就是 FlyBehavior 实
        // 例），该对象是在模型鸭构造器中设置的
        model.performFly();
        // 通过调用 setter 方法，传输新的对象（行为方法），改变原有的行为方法，
        // 即 FlyNoWay() -> FlyRocketPowered()
        model.setFlyBehavior(new FlyRocketPowered());
        // 再次调用 performFly()
        model.performFly();
    }
}
```

### 总结
　　将会变化的部分抽出来，写成行为接口，让子类调用该对象接口的具体行为类。<br />
　　设计原则，多用组合，少用继承。使用组合建立系统具有很大的弹性，不仅将算法族（行为类）封装成类，还可以动态改变行为，只要组合的行为对象符合正确的接口标准即可。

![image.png](attachment:image.png)
