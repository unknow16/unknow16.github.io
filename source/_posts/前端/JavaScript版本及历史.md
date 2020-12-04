---
title: JavaScript版本及历史
date: 2018-05-24 22:15:30
tags: Web
toc: true
---

## ECMAScript主要版本
- 1999年12月 ：ECMAScript 3.0

- 2011年06月 ：ECMAscript 5.1

完善了ECMAScript 3版本、增加"strict mode,"（严格模式）、以及新的功能，如getter和setter、JSON库支持和更完整的对象属性。

- 2015年06月 :  ECMAScript 6 / ES6 / ECMAScript 2015 / ES2015

此版本增加了非常重要的东西：let、const、class、modules、 arrow functions,、template string, destructuring, default, rest argument、binary data、promises等等。

规范地址：http://www.ecma-international.org/ecma-262/6.0/

## ECMA
ECMA是一个组织，前身是欧洲计算机制造商协会

英文全称为European Computer Manufacturers Association

## TC39 
ECMA的39号技术委员会，英文全称 Technical Committee 39

负责制定和审核ECMA-262标准，成员由业内的大公司派出的工程师组成

## ECMAScript
ECMAScript，简称ES，是由ECMA按照ECMA-262标准制定的一种脚本语言规范。

ECMA-262标准后来也被另一个国际标准化组织ISO（International Organization for Standardization）批准，标准号是ISO-16262。


## JavaScript
JavaScript是按ECMAScript规范实现的一种脚本语言，其他的还有JScript、ActionScript。

这三种语言还提供了ECMA规范外的额外功能。



## ECMAScript历史
- 1997年7月，ECMAScript 1.0发布。

- 1998年6月，ECMAScript 2.0版发布。

- 1999年12月，ECMAScript 3.0版发布，成为JavaScript的通行标准，得到了广泛支持。

- 2007年10月，ECMAScript 4.0版草案发布，对3.0版做了大幅升级，预计次年8月发布正式版本。草案发布后，由于4.0版的目标过于激进，各方对于是否通过这个标准，发生了严重分歧。以Yahoo、Microsoft、Google为首的大公司，反对JavaScript的大幅升级，主张小幅改动；以JavaScript创造者Brendan Eich为首的Mozilla公司，则坚持当前的草案。

- 2008年7月，由于对于下一个版本应该包括哪些功能，各方分歧太大，争论过于激进，ECMA开会决定，中止ECMAScript 4.0的开发（即废除了这个版本），将其中涉及现有功能改善的一小部分，发布为ECMAScript 3.1，而将其他激进的设想扩大范围，放入以后的版本，由于会议的气氛，该版本的项目代号起名为Harmony（和谐）。会后不久，ECMAScript 3.1就改名为ECMAScript 5。

- 2009年12月，ECMAScript 5.0版正式发布。Harmony项目则一分为二，一些较为可行的设想定名为JavaScript.next继续开发，后来演变成ECMAScript 6；一些不是很成熟的设想，则被视为JavaScript.next.next，在更远的将来再考虑推出。TC39的总体考虑是，ECMAScript 5与ECMAScript 3基本保持兼容，较大的语法修正和新功能加入，将由JavaScript.next完成。当时，JavaScript.next指的是ECMAScript 6。第六版发布以后，将指ECMAScript 7。TC39预计，ECMAScript 5会在2013年的年中成为JavaScript开发的主流标准，并在此后五年中一直保持这个位置。

- 2011年6月，ECMAscript 5.1版发布，并且成为ISO国际标准（ISO/IEC 16262:2011）。到了2012年底，所有主要浏览器都支持ECMAScript 5.1版的全部功能。

- 2013年3月，ECMAScript 6草案冻结，不再添加新功能。新的功能设想将被放到ECMAScript 7。

- 2013年12月，ECMAScript 6草案发布。然后是12个月的讨论期，听取各方反馈。

- 2015年6月，ECMAScript 6正式发布，并且更名为“ECMAScript 2015”。这是因为TC39委员会计划，以后每年发布一个ECMAScirpt的版本，下一个版本在2016年发布，称为“ECMAScript 2016”。规范地址：http://www.ecma-international.org/ecma-262/6.0/

- 2016年06月, 发布ECMAScript7, 也被称为ECMAScript 2016。完善ES6规范，还包括两个新的功能：求幂运算符（*）和array.prototype.includes方法。规范地址：http://www.ecma-international.org/ecma-262/7.0/

- 2017年06月，发布ECMAScript7, 也被称为ECMAScript 2017。增加新的功能，如并发、原子操作、Object.values/Object.entries、字符串填充、promises、await/asyn等等。规范地址：http://www.ecma-international.org/ecma-262/8.0/

- 除了ECMAScript的版本，很长一段时间中，Netscape公司（以及继承它的Mozilla基金会）在内部依然使用自己的版本号。这导致了JavaScript有自己不同于ECMAScript的版本号。1996年3月，Navigator 2.0内置了JavaScript 1.0。JavaScript 1.1版对应ECMAScript 1.0，但是直到JavaScript 1.4版才完全兼容ECMAScript 1.0。JavaScript 1.5版完全兼容ECMAScript 3.0。目前的JavaScript 1.8版完全兼容ECMAScript 5。

## 参考
http://javascript.ruanyifeng.com/introduction/history.html

https://www.cnblogs.com/polk6/archive/2017/12/05/js-ECMAScript.html