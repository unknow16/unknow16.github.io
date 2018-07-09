---
title: 对注解的思考
date: 2018-07-02 17:05:50
tags: Java
---

## 前言
　　注解（Annotation），实际上和属性、方法一样，都是一个类的组成部分，不过对于初学者来说还是有点陌生的，因为注解是给别人用的，而属性和方法都是自己用的，这就导致没有对注解进行深入的学习，而在使用别人框架的时候，才被迫去了解框架提供的注解的使用方法。

　　注解的形式都是以@开头，在微博微信中@Somebody是通知某人，而注解的@Info，则表示通知一件事（表明一种状态），具体通知谁不去管，具体谁去用也不管，谁对这个注解感兴趣谁来用。
　　
#### 示例
　　我们从最简单的例子说起，最常用的就是@Override，@Deprecated，@SuppressWarnings。如果按照上面的说法来说明，这三个注解都是给编译器使用的。

- 　　@Override：表明这个方法是重写的父类的方法，当你把@Override放到一个方法上时，编译器会自动去父类中查找是否有相应的方法，如果没有，说明注解使用错误，或者重写的方法名、参数等写错了，那么编译器就会给出编译错误，让你去修改。
- 　　@Deprecated：表明这个属性被弃用，当你使用它的时候，编译器就会给出提醒。
- 　　@SuppressWarnings：表明这不是一个警告，那么编译器就不会把它当做警告给提示出来。

#### 再议注解
　　也就是说，注解的使用方便了别人去做某些事情，如果不用注解的话用配置文件也可以，但是针对上面三个注解，如果写在配置文件中，那么编译器要怎么知道去哪个配置文件中去读，又要以怎样的格式去读，这都是一个问题，而使用注解遵从了一种约定大于配置的理念，注解放在类结构的什么的地方，就是给谁的配置，如注解放到类上就是对于类的配置，放到方法上就是给方法的配置，注解能放在类结构的哪些地方在ElementType枚举类中定义。

　　所以使用注解的时候就要明白这个注解是给谁用的，用作什么。

　　而当你打算写一个框架时，也可以提供注解的方式给别人使用，这样来说更方便，而对于你来说就要以解析注解的方式来代替读取并解析配置文件的方式。
　　
## 注解的属性
　　注解也是有相应的属性的，也就是说当定义一个注解的时候指定注解的属性。

#### 注解位置
　　首先了解一下可以被注解的位置有哪些，这些都在一个枚举类：ElementType当中：

- TYPE：类、接口、注解、枚举
- FIELD：字段
- METHOD：方法
- PARAMETER：参数
- CONSTRUCTOR：构造方法
- LOCAL_VARIABLE：本地变量
- ANNOTATION_TYPE：注解
- PACKAGE：包
- TYPE_PARAMETER：类型参数
- TYPE_USE：类型使用

　
注解位置配合@Target使用，当只有一个位置时可以这么使用：

```
@Target(ElementType.ANNOTATION_TYPE)
```

当指定多个位置时，使用方法如下：


```
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
```
如果一个注解没有指定注解位置，那么它可以应用于所有位置。

#### 注解生命周期
　　注解也是有相应的声明周期的，也是封装在一个枚举类：RetentionPolicy中：

- SOURCE：源代码期间，在编译时会去除，所以这都是给编译器使用的
- CLASS：会保留在类文件中，但是运行时JVM不需要保存，默认的生命周期
- RUNTIME：会持续保存到JVM运行时，可以通过反射来获取

声明周期配合@Retention来使用，使用方法如下：

```
@Retention(RetentionPolicy.RUNTIME)
```
一般来说对于编写框架用的注解的生命周期都是RUNTIME。

## 自定义注解
#### 注解定义
　　注解和接口其实很相似，接口里面的方法定义了行为，注解的方法定义了属性，下面先给一个例子：

```
public @interface MyAnnotation {
    //声明属性,可以使用如下类型
    String name();
    String password() default "123";
    int age() default 12;
    TimeUnit gender() default TimeUnit.SECONDS;
    Class<?> clazz();
    int[] arr() default {1,2,3};
    //为了嵌套配置
    Override my2();
}
```
简单总结下：

- 属性是以方法的形式定义的，属性名即为方法名。
- 可以设置默认值，那么当使用该注解的时候，如果不指定该属性则使用默认这
- 没有默认值的，使用时必须赋值
- 属性类型可以为：String、基本类型、Class类型、枚举、注解、以上的一维数组
- 若只有一种属性，且名为value，则赋值时可以不指定属性名
- 同样的只有一个一维数组value[]，则赋值时可以不指定属性名

#### 注解使用
　　当你提供一个注解供别人使用时，那么对方可能将注解应用于允许的位置，并有可能赋值，而我们无法知道注解具体的位置，这时候只能通过反射加遍历的方式来获得注解的位置。下面给出一个例子：
```
// 1. 定义一个默认值的注解
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface DefaultValue{
    String value();
}

// 2. 定义了一个类，指定了一个属性的默认值
class Dog{
    @DefaultValue("little white")
    public String name;
    public int age;
    
    public String toString() {
        return "Dog [name=" + name + ", age=" + age + "]";
    }
}


// 3. 构造一个对象的时候，获取注解并赋默认值
public class UseAnnatation {

    public static void main(String[] args) throws Exception {
        Dog dog=new Dog();
        Field[] fields = Dog.class.getFields();
        for(Field field:fields){
            DefaultValue annotation = field.getAnnotation(DefaultValue.class);
            if(annotation!=null){
                String value = annotation.value();
                field.set(dog, value);
            }
        }
        System.out.println(dog);
    }

}

```
这个例子做了如下事：

1. 定义一个默认值的注解
1. 定义了一个类，指定了一个属性的默认值
1. 构造一个对象的时候，获取注解并赋默认值

当然这个例子只是举例说明注解的用法，默认值根本就不用这么复杂的方式，如果用过Spring的话，应该知道自动注入的注解，实现原理就是通过这种方式。

## JDK已有注解
　　除了一开始说的@Override，@Deprecated，@SuppressWarnings三个注解以外，jdk还有其他注解，简单来说，现在介绍的时候就会贴出源码。

#### @Documented：


```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Documented {
}
```

这个注解用来表明，在生成api文档的时候将注解的对象生成文档。

默认情况下,javadoc是不包括注解的. 但如果声明注解时指定了 @Documented,则它会被 javadoc 之类的工具处理, 所以注解类型信息也会被包括在生成的文档中.

#### @Inherited：


```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited {
}
```

被注解的注解，将会有被子类继承。

#### @Repeatable：


```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Repeatable {
    Class<? extends Annotation> value();
}
```

被注解的注解，可以在一个属性上重复使用。

value属性指示可重复的注解类型	

#### @Native：


```
@Documented
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.SOURCE)
public @interface Native {
}
```

表明一个字段引用的值可能来自于本地代码，暂未找到具体示例，后续补上。

## 总结
　　说到底，注解的使用方还是很简单的，难点在于提供给你注解的人是怎么通过注解去达到他的目的的。如果你设计一个框架并提供注解给人使用，那么你就要精通反射。不过一般情况下是很少遇到需要自定义反射的场景，除非你设计一个中间框架，别人通过你的框架来调用自己实现的类，就像Spring一样。
