
## 模板方法模式（Template Method Pattern）
　　前面已经封装过对象创建、方法调用、复杂接口等，模板方法模式则是封装了算法。<br />
　　如下有两个类，包含重复代码 boilWater() 和 pourInCup()
  
```java
/*
 * 咖啡类的冲泡方法
 */
public class Coffee {
    // 冲泡方法
    void prepareRecipe() {
        boilWater();
        brewCoffeeGrinds();
        pourInCup();
        addSugarAndMilk();
    }
    
    public void boilWater() {
        System.out.println("Boiling water");
    }
    
    public void brewCoffeeGrinds() {
        System.out.println("Dripping Coffee through filter");
    }
    
    public void pourInCup() {
        System.out.println("Pouring into cup");
    }
    
    public void addSugarAndMilk() {
        System.out.println("Adding Sugar and Milk");
    }
}


/*
 * 茶类的冲泡方法
 */
public class Coffee {
    // 冲泡方法
    void prepareRecipe() {
        boilWater();
        steepTeaBag();
        pourInCup();
        addLemon();
    }
    
    public void boilWater() {
        System.out.println("Boiling water");
    }
    
    public void steepTeaBag() {
        System.out.println("Stepping the tea");
    }
    
    public void pourInCup() {
        System.out.println("Pouring into cup");
    }
    
    public void addSugarAndMilk() {
        System.out.println("Adding Lemon");
    }
}
```

　　将重复的代码提取出来，做成超类，会变的部分作为抽象方法，交由子类来实现，如下：
  
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_8/chapter_8_p1.png)

### 抽象 prepareRecipe()
　　将 prepareRecipe 也做抽象处理。

- 咖啡使用 brewCoffeeGrinds() 和 addSugarAndMilk()，而茶使用 steepTeaBag() 和 addSugarAndMilk()，这两个方法功能差不多，于是统一起个名称，比如说 brew() 和 addCondiments()，这样就可以用同一个 prepareRecipe() 方法来处理茶和咖啡；

```java
public abstract class CaffeineBeverage {
    // 模板方法，声明为 final，因为不希望子类覆盖整个方法
    final void prepareRecipe() {
        boilwater();
        brew();
        pourInCup();
        addCondiments();
    }
    // 交由子类实现
    abstract void brew();
    
    abstract void addCondiments();
    
    void boilWater() {
        System.out.println("Boiling water");
    }

    void pourInCup() {
        System.out.println("Pouring into cup");
    }
}
```

- 子类继承超类，实现各自不同的部分，模板方法封装在超类，直接调用即可。

```java
public class Tea extends CaffeineBeverage {
    public void brew() {
        System.out.println("Steeping the tea");
    }
    
    public void addCondiments() {
        System.out.println("Adding Lemon");
    }
}

public class Coffee extends CaffeineBeverage {
    public void brew() {
        System.out.println("Dripping Coffee through filter");
    }
    
    public void addCondiments() {
        System.out.println("Adding Sugar and Milk");
    }
}
```

### 认识模板方法
　　prepareRecipe() 是模板方法，因为它用作一个算法的模板，在这个例子中，算法是用来制作咖啡因饮料的。<br />
　　模板方法定义了一个算法的步骤，并允许子类为一个或多个步骤提供实现。如下：


```java
public abstract class CaffeineBeverage {
    void final prepareRecipe() {
        boilWater();
        brew();
        pourInCup();
        addCondiments();
    }
    
    // 声明为抽象，交由子类继承实现
    abstract void brew():
    
    abstract void addCondiments();
    
    void boilWater() {
        // 实现
    }
    
    void pourInCup() {
        // 实现
    }
}
```

　　模板方法提供了一个框架，通过继承超类，其他子类只需实现抽象方法即可，将代码的复用最大化。CaffeineBeverage 类专注在算法本身，存在于一个地方，容易修改。

### 定义模板方法模式
　　模板方法模式在一个方法中定义一个算法的骨架，将一些步骤通过定义抽象方法延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。<br />
　　这个模式是用来创建一个算法的模板，模板就是一个方法。更具体地说，这个方法将算法定义成一组步骤。其中的任何步骤都可以是抽象的，由子类负责实现，这可以确保算法的结构保持不变，同时由子类提供部分实现。<br />
　　对创建框架来说，模板方法模式时很常见的。由框架控制如何做事情，而由使用框架的人指定框架算法中每个步骤的细节。类图如下：
  
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_8/chapter_8_p2.png)

### 定义抽象类
　　抽象类作为基类，包含由子类实现的抽象方法、具体方法、定义一系列步骤的模板方法。

```java
/**
 * 声明为抽象类，用来作为基类，子类必须实现其操作
 */
public abstract class AbstractClass {
    // 模板方法，声明为 final，以免子类覆盖改变这个算法
    // 模板方法定义了一连串步骤，每个步骤由一个方法代表
    final void templateMethod() {
        primitiveOperation1();
        primitiveOperation2();
        concreteOperation();
        hook();
    }
    
    // 由子类具体实现
    abstract void primitiveOperation1();
    
    abstract void primitiveOperation2();
    // 具体方法，声明为 final，子类无法覆盖，可被模板方法直接使用
    final void concreteOperation() {
        // 实现
    }
    // "默认不做事的方法"，称为 "hook"（钩子），子类可以视情况决定要不要覆盖它们
    void hook() { }
}
```

### 模板方法进行挂钩
　　钩子是一种被声明在抽象类中的方法，但只有空的或者默认的实现。钩子的存在，可以让子类有能力对算法的不同点进行挂钩，要不要挂钩，由子类自行决定。

```java
public abstract class CaffeineBeverageWithHook {
    void prepareRecipe() {
        boilwater();
        brew();
        pourInCup();
        if (customerWantsCondiments()) {
            addCondiments();
        }
    }
        
        abstract void brew();
        
        abstract void addCondiments();
        
        void boilWater() {
            System.out.println("Boiling water");
        }
        
        void pourInCup() {
            System.out.println("Pouring into cup");
        }
        
        // 钩子方法，通常是空的缺省实现，子类可以覆盖整个方法。
        boolean customerWantsCondiments() {
            return true;
        }
}
```

　　当你的子类“必须”提供算法中某个方法或步骤的实现时，就使用抽象方法。如果算法的某个部分是可选的，就用钩子。<br />
　　钩子有几种用法，可以让子类实现算法中可选的部分，或者在钩子对于子类的实现并不重要的时候，子类可以对此钩子置之不理。钩子的另一个用法，是让子类能够有机会对模板方法中某些即将发生的（或刚刚发生的）步骤做出反应。
  
### 好莱坞原则
　　好莱坞原则，别调用我们，我们会调用你。好莱坞原则可以给我们一种防止“依赖腐败”的方法，当高层组件依赖低层组件，而低层组件又依赖高层组件，而高层组件又依赖边侧组件，而边侧组件又依赖低层组件时，依赖腐败就发生了。<br />
　　在好莱坞原则下，允许低层组件将自己挂钩到系统上，但是高层组件会决定什么时候和怎样使用这些低层组件，即高层组件对待低层组件的方式是“别调用我们，我们会调用你”。

### 好莱坞原则和模板方法
　　在设计模板方法模式时，告诉子类，“不要调用我们，我们会调用你”。工厂方法、观察者模式也使用好莱坞原则。<br />
  
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_8/chapter_8_p3.png)

　　好莱坞原则和依赖倒置原则的区别：
  
- 依赖导致原则告诉我们尽量避免使用具体系，而多使用抽象；
- 好莱坞原则是用在创建框架或组件上的一种技巧，好让低层组件能够被挂钩进计算中，而且又不会让高层组件依赖低层组件。两者的目标都是解耦，但依赖倒置原则更加注重如何在设计中避免依赖。

　　**模板方法和工厂方法的区别：**
  
- 模板方法中，子类决定如何实现模板算法中的步骤；
- 工厂方法，由子类决定实例化哪个具体类。

### 用模板方法排序
　　Java 的排序就用到模板方法，mergeSort(Object src[], Object dest[], int low, int high, int off) 为模板方法，其中 compareTo 需要我们自己实现，定义如何比价两个对象的大小。<br />
　　sort() 的设计者希望这个方法能使用于所有的数组（每个数组都是不同的类），并没有使用继承，而是把 sort() 变成静态方法，这样任何数组都可以使用这个方法。<br />
　　还有因为 sort() 并不是真正定义在超类中，所以 sort() 方法需要知道你已经实现了 compareTo() 方法，否则就无法进行排序。要做到这一点，设计者利用 Comparable 接口，需要实现该接口，提供这个接口所声明的方法，即 compareTo()。
   
```java
public static void sort(Object[] a) {
    Ojbect aux[] = (Object[]) a.clone();
    mergeSort(aux, a, 0, a.length, 0);
}

private static void mergeSort(Object src[], Object dest[], int low, int high, int off) {
    for (int i = low; i < high; i++) {
        // 要实现 compareTo() 方法，填补模板方法的缺陷
        for (int j = i; j < low && ((Comparable)dest[j-1].compareTo((Comparable)dest[j]>0; j --) {
        // 具体方法，已经在数组类中定义
        swap(dest, j, j-1);
        }
    }
    return;
}
```

　　与策略模式不同。在策略模式中，所组合的类实现了整个算法，即所有类都实现好，只要组合在一起就可以。而模板方法中有部分是交由子类实现的，数组所实现的排序算法并不完整，需要一个类填补 compareTo() 方法的实现。

### 总结

- 模板方法定义了算法的步骤，并把这些步骤的实现延迟到子类，子类只需实现抽象方法，提供了一种代码复用的重要技巧；
- 模板方法的抽象类可以定义具体方法、抽象方法和钩子，抽象方法由子类实现；
- 钩子是一种方法，它在抽象类中不做事，或者只做默认的事，子类可以选择不去覆盖它；
- 为了防止子类改变模板方法中的算法，可以将模板方法声明为 final；
- 好莱坞元组告诉我们，将决策权放在高层模块中，以便决定如何以及何时调用底层模块；
- 策略模式和模板方法模式都封装算法，一个用组合，一个用继承，而工厂方法是模板方法的一种特殊版本。
