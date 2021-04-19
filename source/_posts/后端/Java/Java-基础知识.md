---
title: Java-基础知识
toc: true
date: 2021-04-19 10:01:43
tags:
categories:
---

## 异常体系
所有异常都是Throwable类的子类，注意Throwable是类不是接口。

Throwable类包含Exception类和Error类，Exception类又包含检查异常和未检查异常。

---
异常定义：程序在运行期间发生的不正常事件，它会打断指令的正常流程，在编译期间产生的叫语法错误。

1. 程序在运行时产生异常时，JVM会创建一个异常对象，交给运行时系统。
2. 运行时系统在抛出异常代码附近寻找处理方式。
3. 两种处理方式：
    * catch: 自己捕获处理
    * throws: 抛给调用者

### 未检查异常/运行时异常（RuntimeException）
对于该异常，Java编译器不要求你一定要捕获或抛出。

RuntimeException的子类都是未检查异常，不需处理。如下：
* NullPointerException
* ClassCastException
* ArrayIndexsOutOfBoundsException
* ArithmeticException(算术异常，除0溢出)

### 检查异常
必须要在方法中捕获或抛出。如：IOException属于检查异常，必须要捕获或再次抛出。

* FileNotFoundException
* IOException
* SQLException


## transient关键字
### transient的作用及使用方法
我们都知道一个对象只要实现了Serilizable接口，这个对象就可以被序列化，java的这种序列化模式为开发者提供了很多便利，我们可以不必关系具体序列化的过程，只要这个类实现了Serilizable接口，这个类的所有属性和方法都会自动序列化。

然而在实际开发过程中，我们常常会遇到这样的问题，这个类的有些属性需要序列化，而其他属性不需要被序列化，打个比方，如果一个用户有一些敏感信息（如密码，银行卡号等），为了安全起见，不希望在网络操作（主要涉及到序列化操作，本地序列化缓存也适用）中被传输，这些信息对应的变量就可以加上transient关键字。换句话说，这个字段的生命周期仅存于调用者的内存中而不会写到磁盘里持久化。

总之，java 的transient关键字为我们提供了便利，你只需要实现Serilizable接口，将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会序列化到指定的目的地中。

示例code如下：

```java
public class TransientTest {

    public static void main(String[] args) throws Exception {
        User user = new User();
        user.setUsername("Alexia");
        user.setPasswd("123456");

        ObjectOutputStream os = new ObjectOutputStream(new FileOutputStream("C:/user.txt"));
        os.writeObject(user); // 将User对象写进文件
        os.flush();
        os.close();

        ObjectInputStream is = new ObjectInputStream(new FileInputStream("C:/user.txt"));
        user = (User) is.readObject(); // 从流中读取User的数据
        is.close();

        System.out.println("read after Serializable: ");
        System.out.println("username: " + user.getUsername()); // Alexia
        System.err.println("password: " + user.getPasswd());  // output null；说明反序列化时根本没有从文件中获取到信息。
    }
}

@Data
class User implements Serializable {
    private static final long serialVersionUID = 8294180014912103005L;

    private String username;
    private transient String passwd;
}
```

### transient说明
- 一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。
- transient关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被transient关键字修饰的。变量如果是用户自定义类，则该类需要实现Serializable接口。
- transient关键字修饰的变量不再能被序列化，一个静态变量不管是否被transient修饰，均不能被序列化。

第三点可能有些人很迷惑，因为发现在User类中的username字段前加上static关键字后，程序运行结果依然不变，即static类型的username也读出来为“Alexia”了，这不与第三点说的矛盾吗？实际上反序列化后类中static型变量username的值为当前JVM中对应static变量的值，这个值是JVM中的不是反序列化得出的。

### transient关键字修饰的变量真的不能被序列化吗？
```java
public class ExternalizableTest implements Externalizable {

    private transient String content = "是的，我将会被序列化，不管我是否被transient关键字修饰";

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(content);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        content = in.readObject().toString();
    }

    public static void main(String[] args) throws Exception {
        ExternalizableTest et = new ExternalizableTest();
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("F:/test.txt"));
        out.writeObject(et);

        ObjectInputStream in = new ObjectInputStream(new FileInputStream("F:/test.txt"));
        et = (ExternalizableTest) in.readObject();
        System.out.println(et.content); // content初始化的内容，而不是null，transient关键字没有起作用

        out.close();
        in.close();
    }
}
```
我们知道在Java中，对象的序列化可以通过实现两种接口来实现，若实现的是Serializable接口，则所有的序列化将会自动进行，若实现的是Externalizable接口，则没有任何东西可以自动序列化，需要在writeExternal方法中进行手工指定所要序列化的变量，这与是否被transient修饰无关。因此第二个例子输出的是变量content初始化的内容，而不是null。

### 部分序列化的实现方式
- 实现Serializable接口，不想序列化的字段使用transient关键字修饰
- 实现Serializable接口，添加writeObject和readObject方法，方法签名在Serializable接口注释中有说明
- 实现Externalizable接口，重写writeExternal和readExternal方法


## 参考资料
> - []()
> - []()
