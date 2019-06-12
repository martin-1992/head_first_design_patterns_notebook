
## 观察者模式（Observer Pattern）
　　观察者模式定义了对象之间的一对多依赖，当一个对象改变状态时，它的所有依赖者都会收到通知并且更新。比如报社为主题对象，订阅报社的用户为观察者，每天报社会把最新的信息（报纸）发给订阅者（观察者），如果 A 用户取消订阅，即从观察者中移除，不再收到报社发的最新信息（报纸）。
  
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_2/chapter_2_p1.png)

### 一对多
　　利用观察者模式，主题是具有状态的对象，并且可以控制这些状态。即，有“一个”具有状态的主题。另一方面，观察者使用这些状态，虽然这些状态并不属于他们，有许多观察者，依赖主题来告诉他们状态何时改变。这就产生一个关系：“一个”主题对应“多个”观察者的关系。
  
![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_2/chapter_2_p2.png)

### 松耦合的威力
　　当两个对象之间松耦合，它们依然可以交互，但是不太清楚彼此的细节。<br />
　　观察者模式提供了一种对象设计，让主题和观察者之间松耦合。主题只知道观察者实现了某个接口，并不需要知道观察者的具体类是谁、做了些什么，任何时候都可以通过实现某个接口来增加新的观察者。主题只会发送通知给所有实现了观察者接口的对象，只要遵守接口，就可自由改变它们。

### 设计气象站

![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_2/chapter_2_p3.png)

- **主题接口，实现该接口，即为主题，可通知观察者；**

```java
public interface Subject {

    /**
     * 注册观察者
     * @param o 观察者对象
     */
    public void registObserver(Observer o);

    /**
     * 删除观察者
     * @param o 观察者对象
     */
    public void removeObserver(Observer o);

    /**
     * 当主题状态改变时，这个方法会被调用，以通知所有的观察者
     */
    public void notifyObservers();
}
```

- **观察者接口，所有观察者必须实现该接口的 update() 方法。因为主题在通知所有观察者时，会调用每个观察者的 update 方法；**

```java
public interface Observer {

    public void update(float temp, float humidity, float pressure);
}
```

- **观察者接口2，这个是用于展示打印收到的推送信息，当调用 update() 方式时，调用 display() 方法；**

```java
/**
 * 当布告板需要显示时，调用此方法
 */
public interface DisplayElement {

    public void display();
}
```

- **主题对象，实现主题接口定的方法；**
    1. 首先创建一个数组；
    2. registObserver 为注册观察者即将观察者对象加入数组；
    3. removeObserver 为删除观察者即从数组中删除对应观察者对象；
    4. setMeasurements，主题参数为最新信息，调用即传入参数，然后通过遍历数组调用每个观察者的 update() 方法来通知更新观察者对象，update() 方法由观察者接口定义，确保主题通过同一个接口方法通知所有观察者。

```java
public class WeatherData implements Subject{

    // 观察者队列
    private ArrayList observers;
    private float temperature;
    private float humidity;
    private float pressure;

    /**
     * 在构造器中创建一个 ArrayList，用于添加、删除观察者
     */
    public WeatherData() {
        observers = new ArrayList();
    }

    /**
     * 注册观察者，即添加到 ArrayList
     * @param o 观察者对象
     */
    public void registObserver(Observer o) {
        observers.add(o);
    }

    /**
     * 当观察者取消注册时，从 ArrayList 中遍历找到该对象并删除
     * @param o 观察者对象
     */
    public void removeObserver(Observer o) {
        int i = observers.indexOf(o);
        if (i >= 0) {
            observers.remove(i);
        }
    }

    /**
     * 主体把状态告诉每一个观察者，因为观察者都实现了 update()，所以我们知道如何通知它
     * 们，不同观察者根据各自实现的 update() 来处理信息，获得不同的结果
     */
    public void notifyObservers() {
        for (int i = 0; i < observers.size(); i++) {
            Observer observer = (Observer) observers.get(i);
            observer.update(temperature, humidity, pressure);
        }
    }

    /**
     * 当从气象站得到更新观测值时，通知观察者
     */
    public void measurementsChanged() {
        notifyObservers();
    }

    /**
     * 主题获得最新信息时，调用 measurementsChanged 通知观察者
     * @param temperature
     * @param humidity
     * @param pressure
     */
    public void setMeasurements(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        measurementsChanged();
    }

    // WeatherData 的其它方法
}

```

- **实现观察者接口，即为观察者对象，可收到主题推送的信息，通过构造器将主题作为参数传入，然后调用主题方法注册观察者；**

```java
/**
 * 此布告板实现了 Observer 接口，所以可以从 weatherData 对象中获得改变
 */
public class CurrentConditionsDisplay implements Observer, DisplayElement {
    private float temperature;
    private float humidity;
    private Subject weatherData;

    /**
     * 通过构造器，注册主题 weatherData 对象
     * @param weatherData
     */
    public CurrentConditionsDisplay(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.registObserver(this);
    }

    /**
     * 当 update() 被调用时，把温度和湿度保存起来，然后调用 display()
     * @param temperature 温度
     * @param humidity 适度
     * @param pressure 压强
     */
    public void update(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        display();
    }

    /**
     * 打印温度和湿度
     */
    public void display() {
        System.out.println("Current conditions: " + temperature +
                "F degress and " + humidity + "% humidity");
    }
}
```

- **测试，首先创建主体对象，然后创建观察者，注册到主体中，即 ArrayList 中。主体通过 ArrayList 遍历每个观察者对象，调用每个观察者的 update() 方法，将最新信息通知给观察者。**

```java
public class WeatherStation {

    public static void main(String[] args) {

        // 创建主体对象
        WeatherData weatherData = new WeatherData();
        // 创建观察者，注册到主体中
        CurrentConditionsDisplay currentConditionsDisplay = new CurrentConditionsDisplay(weatherData);

        // 主体通知观察者，将最新信息推送给观察者
        weatherData.setMeasurements(90, 65, 30.4f);
        weatherData.setMeasurements(82, 70, 30.4f);
        weatherData.setMeasurements(78, 90, 30.4f);
    }
}
```

### 总结

- 观察者模式定义了对象之间一对多的关系；
- 主体（也就是可观察者）用一个共同的接口来更新观察者；
- 观察者和可观察者之间用松耦合方式结合，可观察者不知道观察者的细节，只知道观察者实现了观察者接口；
- 使用此模式时，可从被观察者处推或拉数据。推即主体更新信息，并通知所有观察者。拉则由观察者主动从主体中获得信息；
- 有多个观察者时，不可以依赖特定的通知次序；
- Java 有多种观察者模式的实现，比如 JavaBeans、RMI、MVC 等；
