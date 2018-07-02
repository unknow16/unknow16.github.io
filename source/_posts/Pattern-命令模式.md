---
title: 命令模式
date: 2018-06-13 17:07:24
tags: Pattern
---

### 角色分类
该模式包含下列3个角色：
1. Receiver: 执行具体操作的，操作的具体实现
2. Command：通用命令抽象，定义命令。其中包含Receiver的引用，执行具体操作时，调用Receiver相应方法执行。
3. Invoker: 命令执行者，接收命令，执行命令。其中包括Command的引用，执行具体操作时，调用Command相应方法执行。

### Receiver

```
//通用Receiver类
public abstract class Receiver {
    public abstract void doSomething();
}

//具体Receiver类
public class ConcreteReciver1 extends Receiver{ 
    //每个接收者都必须处理一定的业务逻辑 
    public void doSomething(){ } 
} 
public class ConcreteReciver2 extends Receiver{ 
    //每个接收者都必须处理一定的业务逻辑 
    public void doSomething(){ } 
}
```

### Command

```
//抽象Command类
public abstract class Command {
    public abstract void execute();
}

//具体的Command类
public class ConcreteCommand1 extends Command { 
    //对哪个Receiver类进行命令处理 
    private Receiver receiver; 
    
    //构造函数传递接收者 
    public ConcreteCommand1(Receiver _receiver){
        this.receiver = _receiver; 
    } 

    //必须实现一个命令 
    public void execute() { 
    //业务处理 
        this.receiver.doSomething(); 
    } 
} 

public class ConcreteCommand2 extends Command { 
    //哪个Receiver类进行命令处理 
    private Receiver receiver; 
    //构造函数传递接收者 
    public ConcreteCommand2(Receiver _receiver){
        this.receiver = _receiver; 
    } 
    //必须实现一个命令 
    public void execute() { 
        //业务处理 
        this.receiver.doSomething();
    } 
}
```

### Invoker

```
//调用者Invoker类
public class Invoker {
    private Command command;
    
    public void setCommand(Command _command){
        this.command = _command;
    }
    
    public void action() {
        this.command.execute();
    }
}
```

### 场景类

```
//场景类
public class Client {
    public static void main(String[] args){
        Invoker invoker = new Invoker();
        Receiver receiver = new ConcreteReceiver1();
        
        Command command = new ConcreteCommand1(receiver);
        invoker.setCommand(command);
        invoker.action();
    }
}
```

### 优点
1. 类间解耦

调用者角色与接收者角色之间没有任何依赖关系，调用者实现功能时只需调用Command 抽象类的execute方法就可以，不需要了解到底是哪个接收者执行。

2. 可扩展性

Command的子类可以非常容易地扩展，而调用者Invoker和高层次的模块Client不产生严 重的代码耦合。

3. 命令模式结合其他模式会更优秀

命令模式可以结合责任链模式，实现命令族解析任务；结合模板方法模式，则可以减少 Command子类的膨胀问题。

### 缺点
命令模式也是有缺点的，请看Command的子类：如果有N个命令，问题就出来 了，Command的子类就可不是几个，而是N个，这个类膨胀得非常大，这个就需要读者在项 目中慎重考虑使用。