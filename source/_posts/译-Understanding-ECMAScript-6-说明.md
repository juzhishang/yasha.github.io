title: '[译]Understanding ECMAScript 6 说明'
category:
  - 翻译
tags:
  - Understanding ECMAScript 6
categories:
  - 翻译
date: 2015-11-23 12:07:00
---
### 说明

JavaScript核心语言功能定义在ECMA-262中，此标准定义的语言是ECMAScript，浏览器中的JavaScript和Node.js环境是它的超级。当浏览器与Node.js想要通过额外对象和方法增加更多的功能，其语言核心仍是在ECMAScript中定义的，这就是ECMA-262的持续发展对JavaScript整体成功至关重要的原因。

2007年，JavaScript处于十字路口。Ajax的流行使我们进入了动态web应用的新时代，然而。Javascript自从1999年发布的ECMA-262第三版后就未有变化。 负责推动ECMAScript进程的TC-39委员会, 为ECMAScript 4整合了大量草案规范。ECMAScript 4的范围很大，引入了大大小小的语言变化。 语言特性包含新的语法，模块，类，经典继承，私有对象成员，可选类型注释等等。

ECMAScript 4的范围变化在TC-39内部引发了分歧 , 一些成员认为第四版试图完成的东西太多了。一群来自雅虎、谷歌和微软的带头人为 ECMAScript的下一版本想出了另一个提议，他们最初称之为ECMAScript 3.1。“3.1” 意在表明，这是对现有标准的渐进改变。

ECMAScript 3.1引入了少量语法变化， 不着眼于property 属性，原生JSON 支持, 对已有对象添加方法。 尽管他们早期尝试调和ECMAScript 3.1 与 ECMAScript 4，最终失败了，困难在于这两大阵营在语言该如何成长上有非常不同的观点。

2008年，JavaScript的创造者 Brendan Eich宣布TC-39 会将其努力集中在ECMAScript 3.1的标准化上。他们将搁置 ECMAScript 4中主要的语法和特性变化直到ECMAScript的下一版本标准化，且委员会的所有成员将努力把ECMAScript 3.1 和4的最佳片段整合到一起，之后指向最初的成果称之为 “ECMAScript Harmony”.

ECMAScript 3.1 最终被标准化为ECMA-262的第四版, 也被称为ECMAScript 5。 委员会从未发行过ECMAScript 4标准，以避免与同名的现已不存在的成果混淆。然后开始ECMAScript Harmony的工作，ECMAScript 6成为新“和谐”精神下发行的第一个标准。

2014年，ECMAScript 6达到功能齐全的状态。 功能差别很大，从全新的对象和语法模式变化到现有对象的新方法。令人兴奋的是ECMAScript 6的所有变化都是针对开发者现实面对的问题。虽然采用和实施以达到开发者最低预期的ECMA6仍然会花费时间，对JavaScript的未来是什么样子有好的理解可以使人收获良多。

#### 浏览器与Node.js兼容

许多Javascript环境， 比如浏览器与Node.js，事实上正致力于实现ECMAScript 6。此书并不试图解决实现之间的矛盾，而是着眼于规范定义了什么作为正确的行为。因此，你的JavaScript环境不符合此书描述的行为也是可能的。

#### 这本书是写给谁的

这本书的目的是为那些已经熟悉JavaScript和ECMAScript 5的人作为一个指南。

深入理解这个语言的人没有必要使用此书，它有助于理解ECMAScript 5 与 6的不同之处。特别值得一提的是，这本书是针对中级到高级的想要了解此语言未来的JavaScript开发者（包括浏览器与Node.js环境）。

这本书不适合未写过JavaScript的初学者。要使用这本书，你需要对这个语言有一个良好的基本理解。

#### 概述

+ Chapter 1: 基本知识 介绍了语言中最小的变化。这些新的功能没有引入语法变化，而是 ECMAScript 5之上的增量改变。

+ Chapter 2: 函数 讨论了各种函数的变化。包括箭头函数形式，默认参数，剩余参数等等。

+ Chapter 3: 对象 解释了对象如何创建、修改和使用的变化。主题包括对象直接量语法的改变和新的映射方法。

+ Chapter 4: 类 介绍了JavaScript中类的第一个正式的概念。JavaScript中的类对于那些来自其他语言的开发者来讲，常常会感到困惑。在JavaScript的类语法中加入之后使此语言更加平易近人且更加简洁。

+ Chapter 5: 数组 详述了原生数组的变化和他们可以在JavaScript中使用的新方式。

+ Chapter 6: 迭代器和生成器 讨论了语言添加的迭代器和生成器。 这些特性允许你以强有力的方式使用数据集合，这在JavaScript以前的版本中是不可能的。

+ Chapter 7: 集合 详述了新的集合类型Set, WeakSet, Map, 和WeakMap. 这些类型通过特别为JavaScript设计的添加语义、删除系统垃圾和内存管理补充了数组的用处。

+ Chapter 8: 符号 介绍了符号的概念，定义属性的新方式。符号是一个新的原始类型，可以用来掩盖（但不隐藏）对象的属性和方法。

+ Chapter 9: 代理 讨论了新的代理对象，它允许你拦截对象上执行的每一个操作。 代理给予开发者在对象上的前所未有的控制，比如，在定义新的互动模式上有了无限制的可能性。

+ Chapter 10: Promises 介绍了promises，作为语言的一个新部分。Promises 是基层成果，由于广泛的库支持，最终起飞并得到普及。. ECMAScript 6 规范了promises并使其默认可用。

+ Chapter 11: 模块 详述了JavaScript的官方模块格式。这样做的目的是，这些模块可用替代近年来出现的众多的点对点的模块定义格式。

+ Chapter 12: 模板字符串 讨论了新的内置模板功能。模板字符串旨在以一个安全的方式轻松地创建DSL。

+ Chapter 13: 映射 介绍了正式化的JavaScript映射API。与其他语言类似，ECMAScript 6 映射允许你以精确的级别检查对象，即便你没有创建对象。

#### 帮助与支持

你可以通过访问https://github.com/nzakas/understandinges6 对这本书提出问题，建议更改，和打开pull请求。其他的事请发送消息到邮箱： http://groups.google.com/group/zakasbooks.