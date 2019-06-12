
## 代理模式（Proxy Pattern）
　　所谓代理，就是代表某个真实的对象。在这个案例中，代理就像是糖果机对象一样，但其实幕后是它利用网络和一个远程的真正糖果机沟通。即这个代理假装它是真正的对象，其实一切的动作是它利用网络和真正的对象沟通。
  
### 远程代理的角色
　　远程代理就好比“远程对象的本地代表”，远程对象是活在不同的 Java 虚拟机（JVM）堆中（在不同的地址空间运行的远程对象）。本地代表是一种可以由本地方法调用的对象，其行为会转发到远程对象中。<br />
　　代理假装自己是远程对象，客户调用代理以为自己是在调用远程对象。客户对象所做的就像是在做远程方法的调用，但其实只是调用本地堆中的“代理”，再由代理处理所有网络通信的底层细节。
  
  
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_11/chapter_11_p1.png)

### 定义代理模式
　　代理模式为另一个对象提供一个替身或占位符以控制对这个对象的访问，其中远程代理是一般代理模式的一种实现。<br />
　　使用代理模式创建代表对象，让代表对象控制某对象的访问，被代理的对象可以是远程的对象、创建开销大的对象或需要安全控制的对象。几种代理控制访问的方式：
  
- 远程代理控制访问远程对象；
- 虚拟代理控制访问创建开销大的资源；
- 保护代理基于权限控制对资源的访问。

　　代理模式的类图：

![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_11/chapter_11_p2.png)
　　Proxy 持有 RealSubject 的引用。在某些情况下，Proxy 还负责 RealSubject 对象的创建与销毁。客户和 RealSubject 的交互都必须通过 Proxy，因为 Proxy 和 RealSubject 实现相同的接口，所以在任何用到 RealSubject 的地方，都可以用 Proxy 取代。Proxy 也控制了对 RealSubject 的访问，比如需要某些权限才能访问等。
### 远程代理与虚拟代理的区别

- 远程代理，可以作为另一个 JVM 上对象的本地代表。调用代理的方法，会被代理利用网络转发到远程执行，并且结果会通过网络返回给代理，再由代理将结果转给客户；

![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_11/chapter_11_p3.png)

- 虚拟代理，作为创建开销大的对象代表，直到真正需要一个对象的时候才创建它。当对象在创建前和创建中时，由虚拟代理来扮演对象的替身。对象创建后，代理会将请求直接委托给对象。

![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_11/chapter_11_p4.png)

　　代理模式有很多变体，但都有共通点，都会将客户对主题（Subject）施加的方法调用拦截下来。这种间接的级别可以做其它事，比如将请求分发到远程主题，给创建开销大的对象提供代表，提供某些级别的保护（决定哪些客户能调用哪些方法）。<br />
  
### 装饰者模式和代理模式的区别
　　装饰者模式和代理模式都是使用继承同一个接口方法，看起来都是用一个对象把另一个包起来，然后把调用委托给装饰（或代理）对象。主要区别在于两者的目的不一样，装饰者为对象增加行为，而代理是控制对象的访问：

- 装饰者模式是装饰对象，为对象增加行为，改变对象的行为，而代理模式是控制对象的访问；
- 代理模式可保护对象避免不想要的访问，避免在加载大对象的过程中 GUI 会挂起，或者隐藏主题在远程运行的事实，因为客户访问的是真正主题（Subject） 的替身；
- 远程代理中，所代表的和控制访问的对象不再一台机器上。

### 动态代理
　　动态代理，实际的代理类是在运行时创建的。类图如下：
    
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_11/chapter_11_p5.png)

　　InvocationHandler 的工作是响应代理的任何调用，InvocationHandler 想成是代理收到方法调用后，请求做实际工作的对象。

　　现在要创建一个保护代理实例，以实现约会服务系统为例，在服务中加入了喜欢（Hot）和不喜欢（Not），要求在于顾客不可以改变自己的 HotOrNot 评分，也不可以改变其他人的信息。<br />
　　保护代理，是一种根据访问权限决定客户可否访问对象的代理，比如说，如果有一个雇员对象，保护代理允许雇员调用对象上的某些方法，经理可以多调用一些其他的方法，而人力资源处的雇员可以调用对象上的所有方法等。
  
- 首先创建两个 InvocationHandler，实现了代理的行为，其中一个给拥有者使用，另一个给非拥有者使用。InvocationHandler 理解为当代理的方法被调用时，代理会把这个调用转发给 InvocationHandler。注意，无论代理代理调用哪种方法，InvocationHandler 调用都是 invoke() 方法，即使用反射；

```java
// 假设 proxy 的 setHotOrNotRating() 方法被调用
proxy.setHotOrNotRating(8);
// proxy 会接着调用 InvocationHandler 的 invoke() 方法
invoke(Object proxy, Method method, Object[] args);

/**
 * 所有调用处理器都实现 InvocationHandler 接口
 */
import java.lang.reflect.*;

public class OwnerInvocationHandler implements InvocationHandler {
    PersonBean person;
    
    // 通过构造器，将 person 传入
    public OwnerInvocationHandler(PersonBean person) {
        this.person = person;
    }
    
    public Object invoke(Object proxy, Method method, Object[] args) throws IllegalAccessException {
        
        try {
            // 通过反射来调用方法，判断要调用的方法，如果为 setHotOrNotRating，则抛出异常
            if (method.getName().startsWith("get") {
                return method.invoke(person, args);
            } else if (method.getName().equals("setHotOrNotRating")) {
                throw new IllegalAccessException();
            } else if (method.getName().startsWith("set") {
                return method.invoke(person, args);
            }
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
        // 调用其它方法，则返回 null
        return null;
        }
}
```

- 创建 Proxy 类并实例化 Proxy 对象，编写一个以 PersonBean 为参数，即创建一个代理，将它的方法调用转发给 OwnerInvocationHandler。

```java
/**
 * 此方法需要一个 person 对象作为参数，然后返回它的代理，
 * 因为代理和主题有相同的接口，所以返回一个 PersonBean
 */
PersonBean getOwnerProxy(PersonBean person) {
    // 创建代理，利用 Proxy 类的静态 newProxyInstance 方法创建代理，
    // 将 personBean 的类载入器当做参数，
    return (PersonBean) Proxy.newProxyInstance(
        person.getClass().getClassLoader(),
        person.getClass().getInterfaces(),
        // 将 person 传入调用处理器的构造器中，这是处理器能够访问真实主题的原因
        new OwnerInvocationHandler(person));
}
```

- 测试代码，如下将一个对象传入动态代理 getOwnerProxy 或 getNonOwnerPorxy 中，以 getOwnerProxy 为例会调用 OwnerInvocationHandler 处理器来处理，该代码会检测调用的方法，如果为 setHotOrNotRating，则抛出异常，即对对象调用的方法受限，同理 getNonOwnerPorxy 也一样。

```java
public class MatchMakingTestDrive {
    // 这里有实例变量
    
    public static void main(String[] args) {
        MatchMakingTestDrive test = new MatchMakingTestDrive();
        test.drive();
    }
    
    public MatchMakingTestDrive() {
        initializeDatabase();
    }
    
    public void drive() {
        PersonBean joe = getPersonFromDatabase("Joe Javabean");
        // 创建一个代理
        PersonBean ownerProxy = getOwnerProxy(joe);
        // 调用 getter 和 setter 方法
        System.out.println("Name is " + ownerProxy.getName());
        ownerProxy.setInterests("bowling, Go");
        try {
            // 改变评分方法是不行的，因为代理检测并设置该方法不允许
            ownerProxy.setHotOrNotRating(10):
        } catch (Exception e) {
            System.out.println("can't set rating");
        }
    }
}
```

　　动态代理之所以被称为动态，是因为运行时才将它的类创建出来，代码开始执行时，还没有 proxy 类，它是根据需要从你的接口集创建的。

### 总结

- 代理模式为另一个对象提供代表，以便控制客户对对象的访问，管理访问的方式有许多种；
- 虚拟代理控制访问实例化开销大的对象；
- 保护代理基于调用者控制对对象方法的访问；
- 代理在结构上类似装饰者，但目的不同，装饰者为对象加上行为，而代理则是控制访问；
- 和其它装饰者一样，代理会造成你的设计中类的数目增加。
