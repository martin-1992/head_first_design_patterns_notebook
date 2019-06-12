
## 命令模式（Command Pattern）
　　封装调用，即把方法调用封装起来，调用此运算的对象不需要关心事情是如何进行的，只要知道如何使用包装成形的方法来完成即可。<br />

### 第一个命令对象

- 让所有命令对象实现相同的包含一个方法的接口，方法名一般为 execute() ，如下为命令接口；

```java
public interface Command {
    public void execute();
}
```

- 实现一个命令对象的方法，以电灯为例，打开电灯为命令，通过实现 Command() 接口，将其封装在 execute() 方法中，用户不需要知道该对象是怎么运作的，只需传入命令对象，让后调用封装好的命令方法即可；

```java
/*
 * 这是一个命令，需要实现 Command 接口
 */
public class LightOnCommand extend Command {
    Light light;
    
    // 传入命令对象，记录在实例变量中
    public LightOnCommand(Light light) {
        this.light = light;
    }
    
    // 封装调用命令对象的方法
    public void execute() {
        light.on();
    }
}
```

- 使用命令对象，假设有一个遥控器，只有一个按钮和对应插槽，可以控制一个装置；

```java
public class SimpleRemoteControl {
    // 一个插槽持有命令，这个命令控制着一个装置
    Command slot;
    
    public SimpleRemoteControl() { };
    
    // 插槽装置命令对象
    public void setCommand(Command command) {
        slot = command;
    }
    
    // 按下按钮，调用插槽上的命令对象的命令
    public void buttonWasPressed() {
        slot.execute();
    }
    
}
```

- 测试遥控器代码。

```java
public class RemoteControlTest {
    public static void main(String[] args) {
        // 遥控器为调用者，会传入一个命令对象，可以用来发出请求
        SimpleRemoteControl remote = new SimpleRemoteControl();
        // 创建电灯对象，为请求的接收者
        Light light = new Light();
        // 创建一个命令，将接收者传给它
        LightOnCommand lightOn = new LightOnCommand(light);
        
        // 把命令传给调用者
        remote.setCommand(lightOn);
        // 调用命令的方法，封装好的
        remote.buttonWasPressed();
    }
}
```

- 还可以创建一组命令对象数组，命令对象数组中的每个位置都预先设置为空对象。当你不想返回一个有意义的对象时，或没有设置命令对象时，使用空对象，它的 execute() 方法是什么都不用做

```java
// 空对象，什么都不做
public class NoCommand implements Command {
    public void execute();
}


public class RemoteControl {
    Command[] onCommands;
    Command[] offCommands;
    
    // 构造器，实例化并初始两个开关数组
    public RemoteControl() {
        onCommands = new Command[2];
        offCommands = new Command[2];
        // 默认的命令对象
        Command noCommand = new NoCommand();
        // 预先填充命令对象
        for (int i = 0; i < 2; i++) {
            onCommands[i] = noCommand;
            offCommands[i] = noCommand;     
        }     
     }
     
     // 三个参数，分别为插槽位置、开的命令、关的命令
     public void setCommand(int slot, Command onCommand, Command offCommand) {
         onCommands[i] = onCommand;
         offCommands[i] = offCommand;    
     }
     
     // 封装，调用方法
     public void onButtonWasPushed(int slot) {
         onCommands[slot].execute();
     }
     
     public void offButtonWasPushed(int slot) {
         offCommands[slot].execute();
     }
}
```

### 定义命令模式
　　命令模式将“请求”封装成对象，以便使用不同的请求、队列或者日志来参数化其它对象，命令模式也支持可撤销的操作。<br />
　　一个命令对象通过在特定接收者上绑定一组动作来封装一个请求，命令对象将动作和接收者包进对象中。这个对象只暴露出一个 execute() 方法，当此方法被调用时，接收者就会进行这些动作。从外来看，其他对象不知道哪个接收者进行了哪些动作，只知道调用 execute() 方法，请求的目的就能达到。<br />
  
### 命令模式类图

![Aaron Swartz](https://raw.githubusercontent.com/martin-1992/head_first_design_patterns_notebook/master/chapter_6/chapter_6_p1.png)

### 宏命令
　　将一组命令对象放在一个数组里，然后遍历执行 execute() ，既可将命令对象全部执行。
    
```java
public class MacroCommand implements Command {
    Command[] commands;
    
    // 在宏命令中，用命令数组存储一大堆命令
    public MacroCommand(Command[] commands) {
        this.commands = commands;
    }
    
    // 一次性执行数组里的每个命令
    public void execute() {
        for (int i = 0; i < commands.length; i++) {
            commands[i].execute();
        }
    }
}
```

### 命令模式用在队列请求
　　命令可以将运算块打包（一个接收者 setXX 和一个动作如 execute），像对象一样可将它传来传去，可在不同线程中被调用，比如日程安排、线程池、工作队列等。<br />
　　想象有一个工作队列，在某一端添加命令，另一端则是线程。从队列中取出一个命令，调用它的 execute() 方法，等待这个调用完成，然后将此命令对象丢弃，在取出下一个命令对象，调用它的 execute() 方法，以此类推。<br />
　　工作队列类和进行计算的对象之间是完全解耦的，工作队列不在乎命令对象做什么，只是取出命令对象，然后执行 execute() 方法，只要是实现命令模式的对象，就可以放到队列里。
  
### 命令模式用在日志请求
　　某些应用需要将所有的动作都记录在日志中，并能在系统死机后，重新调用这些动作恢复到之前的状态。通过新增两个方法（store()、load()），命令模式能够支持这一点。<br />
　　当执行命令时，将历史记录存储在磁盘中。一旦系统死机，可以将命令对象重新加载，并成批地依次调用这些对象的 execute() 方法。

### 总结

- 命令模式将发出请求的对象和执行请求的对象解耦，不关心对象是谁，只要能执行 exeucte() 方法即可；
- 在被解耦的两者之间是通过命令对象进行沟通的，命令对象封装了接收者和一个或一组动作；
- 调用者通过调用命令对象的 execute() 发出请求，这会使得接收者的动作被调用；
- 调用者可以接收命令当做参数，甚至在运行时动态进行；
- 命令可以支持撤销，做法是实现一个 undo() 方法来回到 execute() 被执行前的状态；
- 宏命令是命令的一种简单延伸，允许调用多个命令，宏方法也可以支持撤销；
- 实际操作时，很常见使用“聪明”命令对象，也就是直接实现了请求，而不是将工作委托给接收者；
- 命令也可以用来实现日志和事务系统。
