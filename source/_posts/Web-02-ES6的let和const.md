---
title: Web-02-ES6的let和const
date: 2018-08-05 19:18:25
tags: Web
---

## let的作用
用于声明一个块级作用域的变量。

如在if()语句内用let声明的变量，不会延伸到if()语句外部。
```
function foo() {
    if (true) {
        let x = 1;
    }
    console.log(x); // error：x is not defined
}
foo();
```

## let与var的区别
#### 0. 在比较两者区别之前，首先需要了解var的作用域和变量提升：
1. var没有块级作用域
2. 在函数体中任何地方用var声明的变量都相当于在函数体第一行声明的变量，在全局内任何地方声明变量相当于在其内部最顶上声明它，这种行为称为Hoisting（提升）。


#### 1. 作用域区别
- var：没有块级作用域，因为变量提升的特性，其声明变量的作用域为整个函数或全局范围。

- let：拥有块级作用域的特性，其声明的变量作用域范围从声明处一直到当前块级语句(若存在)的结尾，否则会一直延伸到函数结尾(在函数内)或全局结尾。

#### 2. 在全局作用域声明的变量是否作为window对象的成员
- var：在全局作用域声明的变量，会作为widnow对象的成员。

- let：在全局作用域声明的变量，不会作为window对象的成员。
```
var x = 1;
let y = 1;
console.log(window.x); // => 1
console.log(window.y); // => undefined:let声明的变量没有附加到window对象上
```
#### 3. 同级作用域声明同名变量的区别
在在同一个作用域声明2个名称一样的变量时两者的表现：

- var：因变量提升特性，声明2个相同的变量没问题。

- let：报错，会提示变量已存在。
```
function foo() {
    var x = 1;
    let x = 2; // error:Identifier 'x' has already been declared
}
foo();
```
#### 4. for()循环语句内延迟输出循环变量
在下面代码中使用setTimeout函数延迟输出let和var定义的变量：
```
function foo() {
    for (var x = 0; x < 5; x++) {
        setTimeout(function() {
            console.log(x); // => 5 5 5 5 5
        }, 100);
    }
 
    for (let y = 0; y < 5; y++) {
        setTimeout(function() {
            console.log(y); // => 0 1 2 3 4
        }, 100);
    }
}
foo();
```
可以看到使用var定义的x变量将会输出5次5，而使用let定义的y变量会依次输出0 1 2 3 4，这是为什么呢？

还是因为var的变量提升特性，第一个循环体的变量x，实际就为1个；而第二个循环体，每循环一次创建一次变量y。

所以上面的代码可转换以下代码：
```
function foo() {
    var x;
    for (x = 0; x < 5; x++) {
        setTimeout(function() {
            console.log(x); // => 5 5 5 5 5
        }, 100);
    }
 
    for (let y = 0; y < 5; y++) {
        setTimeout(function() {
            console.log(y); // => 0 1 2 3 4
        }, 100);
    }
}
foo();
```

## Babel对let的处理
let是属于ES6，当使用Babel将其转换为ES5的代码是时是怎么转换的呢？可得出几种转换场景：

1. 在同一作用域范围内，若 let 前面没有用 var 定义过的同名变量时，直接使用 var 代替 let 用于声明变量
```
if(ture){
    let x = 2;
}
// 转换后：
if(true){
    var x = 2;
}

```
2. 在同一作用域范围内，若 let 前面出现过用 var 定义同名变量时，修改 let 声明的变量名，并用 var 代替声明
```
var x = 1;
if(true){
    let x = 2;
}
// 转换后：
var x = 1;
if(true){
    var _x = 2;
}
```
3. 若在一个循环语句内部，let声明的变量 参与了 延时函数(setTimeout、setInterval)的执行时，那么延时函数会转换一个独立函数
```
for (x = 0; x < 5; x++) {
    setTimeout(function() {
        console.log(x); // => 5 5 5 5 5
    }, 100);
}
// 转换后：
var _loop = function _loop(x) {
    setTimeout(function (){
        console.log(x);
    }, 100);
};

for (var x = 0; x < 5; x++) {
    _loop(x);
}
```

## const的作用
const用于定义一个常量变量。

1. 作用域的范围与let一样，声明前无法使用以const声明的常量。

1. 同一作用域范围内，const不能声明同名常量。
 
1. const声明的常量不会成为window对象的成员。

1. 当用const声明的常量为值类型(e.g. String、Number)时，修改此常量的值会报错；但当声明的常量为引用类型(e.g. Array、Object)时，只可以修改此常量的成员。

#### 修改值类型的常量会报错

```
const x = 1;
x = 2; // => Uncaught TypeError: Assignment to constant variable.
```

#### 修改引用类型的常量的成员不会报错
```
// 1.const声明一个数组
const x = [1, 2, 3];
console.log(x); // => [1, 2, 3]
x[0] = 2; // 修改数组的第一个元素的值
console.log(x); // => [2, 2, 3]
 
// 2.const声明一个对象
const obj = {};
obj.name = 'polk';
console.log(obj.name); // => polk
```
