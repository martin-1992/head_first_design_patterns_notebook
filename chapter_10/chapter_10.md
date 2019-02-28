
## 状态模式（State Pattern）
　　糖果机的控制器需要如下图般工作，需要设计糖果机。

![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_10/chapter_10_p1.png)

　　这里使用状态模式来设计，方便添加新状态。
  
- 定义一个 State 接口，在这个接口内，糖果机的每个动作都有一个对应方法；
- 为机器中的每个状态实现状态类，这些类将负责在对应的状态下进行机器的行为；
- 将动作委托给状态类。

### 定义状态接口和类
　　创建一个 State 接口，所有的状态都必须实现这个接口，每个方法直接映射到糖果机上可能发生的动作。然后将设计中的每个状态都封装成一个类，每个都实现 State 接口。
  
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_10/chapter_10_p2.png)

### 实现状态类
　　实现适合所在这个状态的行为，在某些情况下，这个行为会让糖果机的状态改变。

```java
/**
 * 需要实现 State 接口
 */
public class NoQuarterState implements State {
    GumballMachine gumballMachine;
    
    // 通过构造器得到糖果机的引用，将其记录在实例变量中
    public NoQuarterState(GumballMachine gumballMachine) {
        this.gumballMachine = gumballMachine;
    }
    
    // 投了 25 分钱，就打印出一条消息，说接受了 25 分钱，然后改变机器的状态到 HasQuarterState
    public void insertQuarter() {
        System.out.println("You inserted a quarter");
        gumballMachine.setState(gumballMachine.getHasQuarterState());
    }
    
    // 没给钱，就不能要求退钱
    public void ejectQuarter() {
        System.out.println("You haven't inserted a quarter");
    }
    
    // 没给钱，就不能要求糖果
    public void turnCrank() {
        System.out.println("You turned，but there's no quarter");
    }
    
    // 没得到钱，就不能发放糖果
    public void dispense() {
        System.out.println("You need to pay first");
    }
    
}
```

### 重新改造糖果机
　　所有的状态对象都是在构造器中创建并赋值的，这个实例变量现在持有一个状态对象，而不是一个整数。

```java
public class GumballMachine {
    State soldOutState;
    State noQuarterState;
    State hasQuarterState;
    State soldState;
    
    State state = soldOutState;
    // 记录机器内装有多少糖果，开始机器是没有装糖果
    int count = 0;
    
    // 构造器取得糖果的初始数目并把它存放在一个实例变量中
    public GumballMachine(int numberGumballs) {
        // 每一种状态都创建一个状态实例
        soldOutState = new SoldOutState(this);
        noQuarterState = new NoQuarterState(this);
        hasQuarterState = new HasQuarterState(this);
        soldState = new SoldState(this);
        this.count = numberGumballs;
        // 糖果数目大于 0，就把状态设置为 NoQuarterState
        if (numberGumballs > 0) {
            state = noQuarterState;
        }
    }
    
    public void insertQuarter() {
        state.insertQuarter();
    }
    
    public void ejectQuarter() {
        state.ejectQuarter();
    }
    
    // 发放糖果是跟在投硬币状态后，不用单独设置
    public void turnCrank() {
        state.turnCrank();
        state.dispense();
    }
    
    // 允许其他的对象将机器的状态转换到不同的状态
    void setState(State state) {
        this.state = state;
    }
    
    // 释放糖果，糖果数 - 1
    void releaseBall() {
        System.out.println("A gumball comes rolling out the slot...");
        if (count != 0) {
            count = count - 1;
        }
    }
    
    // 更多方法，包括 setter() / getter() 方法
}
```

### 实现更多的状态

```java
/**
 * 投入硬币
 */
public class HasQuarterState implements State {
    GumballMachine gumballMachine;
    
    // 当状态被实例化的时候，要传入 GumballMachine 的引用作为参数
    public HasQuarterState(GumballMachine gumballMachine) {
        this.gumballMachine = gumballMachine;
    }
    
    
    // 这是一个对此状态不恰当的动作
    public void insertQuarter() {
        System.out.println("You can't insert another quarter");
    }
    
    // 退出顾客的 25 分钱，并将状态转换到 NoQuarterState 状态
    public void ejectQuarter() {
        System.out.println("Quarter returned");
        gumballMachine.setState(gumballMachine.getNoQuarterState());
    }
    
    // 设置机器状态，转为 SoldState 状态
    public void turnCrank() {
        System.out.println("You turned...");
        gumballMachine.setState(gumballMachine.getSoldState());
    }
    
    // 不恰当动作
    public void dispense() {
        System.out.println("No gumball dispensed");
    }
}
```

```java
/**
 * 出售状态
 */
public class SoldState implements State {
    GumballMachine gumballMachine;
    
    // 当状态被实例化的时候，要传入 GumballMachine 的引用作为参数
    public SoldState(GumballMachine gumballMachine) {
        this.gumballMachine = gumballMachine;
    }
    
    // 不恰当的动作
    public void insertQuarter() {
        System.out.println("Please wait, we're already giving you a gumball");
    }
    
    // 不恰当的动作
    public void ejectQuarter() {
        System.out.println("Sorry, you already turned the crank");
    }
    
    // 不恰当的动作
    public void turnCrank() {
        System.out.println("Turning twice does'nt get you another gumball");
    }
    
    // 发放糖果，检查糖果的剩余数目，然后将状态转换到 NoQuaterState 或 SoldOutState
    public void dispense() {
        gumballMachine.releaseBall();
        
        if (gumballMachine.getCount() > 0) {
            gumballMachine.setState(gumballMachine.getNoQuaterState());
        } else {
            System.out.println("out of gumballs");
            gumballMachine.setState(gumballMachine.getSoldOutState());
        }
    }
}
```

　　糖果机的实现，通过从结构上改变实现：

- 将每个状态的行为局部到它自己的类中；
- 将容易产生问题的 if 语句删除，以方便日后的维护，比如 if 投币，则该如何等等；
- 让每一个状态“对修改关闭”，让糖果机“对扩展开放”，这样可以加入新的状态类；
- 创建一个新的代码基和类结构，这更能映射糖果公司的图，而且更容易阅读和理解。

### 定义状态模式
　　状态模式允许对象在内部状态改变时改变它的行为，对象看起来好像修改了它的类，实际上是使用组合通过简单引用不同的状态对象来造成类改变的假象。这个模式将状态封装成为独立的类，并将动作委托到代表当前状态的对象，行为会随着内部状态而改变。<br />
　　以前面提到的糖果机为例，当糖果机是在 NoQuarterState 或 HasQuarterState 两种不同的状态时，投入 25 分钱，就会得到不同的行为（机器接受 25 分钱或拒绝 25 分钱）。<br />
　　状态模式的类图：

![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_10/chapter_10_p3.png)

　　策略模式和状态模式的区别：
  
- 意图不一样。策略模式和状态模式的类图一样，但这两个模式的差别在于它们的“意图”。以状态模式而言，将一群行为封装在状态对象中，context 的行为随时可以委托到那些状态对象中的一个；
- 状态模式的状态类会改变，但策略模式的行为类一般是指定的，不会改变。当前状态可以在状态对象集合中改变，反映出 context 内部的状态，随着状态的改变，context 的行为也会跟着改变，客户对于状态对象了解不多，甚至不知道。对于策略模式，客户通常主动指定 Context 所要组合的策略对象是哪一个，对于某个 context 对象来说，通常都只有一个最适当的策略对象。比如第一章里，有些鸭子调用会飞行的行为类，不像状态模式，这个飞行行为类会转变成不会飞行的类；
- 策略模式是除继承外的弹性替代方案，状态模式是不用在 context 中放置许多条件判断的替代方案。当使用继承定义了一个类的行为，如果要修改它很难，则使用策略模式将会改变的部分抽取出来做成接口，实现另外一个具体类供调用。通过策略模式，组合不同的对象来改变行为。状态模式则通过将行为包装进状态对象中，可以通过在 context 内简单地改变状态对象来改变 context 的行为。

### 总结

- 状态模式允许一个对象基于内部状态而拥有不同的行为；
- 和程序状态机（PSM）不同，状态模式用类代表状态；
- Context 会将行为委托给当前状态对象；
- 通过将每个状态封装进一个类，把以后需要做的任何改变局部化了，取代了多个条件判断，通过使用 setter()，可将状态 A 转为状态 B；
- 策略模式通常会用行为或算法来配置 Context 类，状态模式运行 Context 随着状态的改变而改变行为；
- 状态转换可以由 State 类或 Context 类控制；
- 使用状态模式通常会导致设计中类的数目大量增加，可假定为有多少个 if 条件就有多少个状态；‘
- 状态类可以被多个 Context 实例共享。
