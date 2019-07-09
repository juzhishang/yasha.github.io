title: '[译]Understanding ECMAScript6 函数'
date: 2015-11-23 19:21:26
tags:
---
### 函数

函数是任何编程语言的重要组成部分，而自从JavaScript被引入以来，JavaScript的函数就未有太多改变。遗留下来的积压问题及微妙行为使我们很容易犯错误，或者需要更多的代码来实现一个非常常见的行为。

ECMAScript6的函数是一个巨大的进步，它考虑了javascript开发者多年的抱怨与请求。结果就是大量的在ECMAScript5之上的函数的增量改进使得javascript编程更不容易出错且比以往更加强大。

#### 默认参数

javascript函数的独特之处在于它们允许传递任意数量的参数，不管在函数定义中声明的参数数量是多少。这允许你定义可以处理不同参数数量的函数，当提供参数时往往只是覆盖默认值。在ECMAScript5及以前，你可能会使用以下模式来完成：

```javascript
function makeRequest(url, timeout, callback) {

    timeout = timeout || 2000;
    callback = callback || function() {};

    // the rest of the function
}
```
此例中，timeout和callback事实上都是可选的因为如果不提供的话它们都会给出默认值。当第一个操作数为假时，逻辑OR操作总是返回第二个操作数。因为有名字的未明确提供值的函数参数被设置为undefined，逻辑OR操作往往被用来为缺省的参数提供默认值。然而，这个方法的瑕疵在于timeout的有效值可能是0，但是它会被2000替代因为0是价值。

其他确定参数缺省的方法包括arguments.length检测传递的参数数量或者直接检测每一个参数看是不是undefined。

ECMAScript6通过为参数提供默认值使传递默认参数变得更加容易，当参数未正式传递时使用初始化的值。比如：
```javascript
function makeRequest(url, timeout = 2000, callback = function() {}) {

    // the rest of the function
}
```
这里，只有第一个参数被要求总是传递。另两个参数有默认值，它使函数体更小因为你再也不需要添加任何代码来检测缺省值。当带三个参数的makeRequest()被调用时，不使用默认值。比如：
```javascript
// uses default timeout and callback
makeRequest("/foo");

// uses default callback
makeRequest("/foo", 500);

// doesn't use defaults
makeRequest("/foo", 500, function(body) {
    doSomething(body);
});
```
任何带默认值的参数被认为是可选的参数，而那些没有默认值的被认为是必填参数。

对任何参数指定默认值都是可能的，包括那些在没有默认值的参数之前的参数。比如，这样也行：
```javascript
function makeRequest(url, timeout = 2000, callback) {

    // the rest of the function
}
```

这种情况，timeout的默认值只有在没有第二个参数传入时或者第二个参数明确传入了undefined时才会被使用。比如：
```javascript
// uses default timeout
makeRequest("/foo", undefined, function(body) {
    doSomething(body);
});

// uses default timeout
makeRequest("/foo");

// doesn't use default timeout
makeRequest("/foo", null, function(body) {
    doSomething(body);
});
```
在有默认参数值的情况中，值null被认为有效于是默认值就不会被使用。

也许默认参数最有意思的功能在于默认值不必是一个原始值，比如，你可以执行一个函数来重新得到默认参数：
```javascript
function getCallback() {
    returnfunction() {
        // some code    };
}

function makeRequest(url, timeout = 2000, callback = getCallback()) {

    // the rest of the function
}
```
这里，如果未提供最后一个参数，函数getCallback()被调用以获取正确的默认值。这给动态将信息注入函数带来了许多可能性。

#### 剩余参数

既然JavaScript函数可以传入任何数量的参数，就不需要总是具体定义每个参数了。在早期的时候，JavaScript提供了arguments对象来作为未单独定义每一个传入的参数时检测所有函数参数的一种方式。大多数情况下它做得很好，但它也会成为工作时的小麻烦，比如：
```javascript
function sum(first) {
    let result = first,
        i = 1,
        len = arguments.length;

    while (i < len) {
        result += arguments[i];
        i++;
    }

    return result;
}
```
这个函数将所有传递给它的参数相加，所以你调用sum(1)或者sum(1,2,3,4)它都会运行。关于函数，有几件事情需要注意。首先，函数能够处理多个参数并不是显而易见的。你可以添加一些更多的命名参数，但你总是缺少这个函数可以获取任意数量参数的提示。第二，因为第一个参数被命名且直接使用，你不得不在查找arguments对象时以1开始而不是以0开始。记得使用合适的arguments索引并不一定是困难的，但还有一件事是跟踪。ECMAScript6引入多余参数来帮助解决这些问题。  

 剩余参数由命名参数前加三个点(...)标示。这个命名参数于是变成了一个包含了剩余参数的数组（这也是为什么它们被称为“剩余”参数）。比如，sum()可以用剩余参数重写如下：
```javascript
function sum(first, ...numbers) {
    let result = first,
        i = 0,
        len = numbers.length;

    while (i < len) {
        result += numbers[i];
        i++;
    }

    return result;
}
```
这个版本的函数中，numbers是一个剩余参数，它包含了第一个之后的所有参数（不像arguments，包含包括第一个在内的所有参数）。这意味着你可以不用担心的从开始到结束遍历numbers。额外令人高兴的是，你一看就知道函数能够处理任意数量的参数。

注意：sum()方法实际上并不需要任何命名参数。理论上，你可以只使用剩余参数，有它仍然会做完全一样的工作。然而，这种情况下，剩余参数实际上就同arguments一样了，所以你并没有真正得到任何额外的好处。

剩余函数唯一的限制是，在函数声明时它后面不能跟随任何其他的命名参数。比如，这会引起语法错误：
```javascript
// Syntax error: Can't have a named parameter after rest parametersfunction sum(first, ...numbers, last) {
    let result = first,
        i = 0,
        len = numbers.length;

    while (i < len) {
        result += numbers[i];
        i++;
    }

    return result;
}
```
这里，参数last紧随在剩余参数numbers之后，导致语法错误。

剩余参数在JavaScript中是设计用来取代arguments的。原来的ECMAScript4取消了arguments然后增加了剩余参数以允许传递无限制数量的参数给函数。尽管ECMAScript4从未产生过，这个想法被保留了下来，而后重新在ECMAScript6中被引入，尽管arguments在此语言中未被移除。

#### 解构参数

第一章中，你了解了解构赋值。解构也可以在赋值表达式上下文的外面使用，也许，最有趣的是有解构参数的情况。

需要大量可选参数的函数使用一个或更多可选的参数对象是很常见的，比如：
```javascript
function setCookie(name, value, options) {

    options = options || {};

    var secure = options.secure,
        path = options.path,
        domain = options.domain,
        expires = options.expires;

    // ...}

setCookie("type", "js", {
    secure: true,
    expires: 60000
});
```
在JavaScript库中，有许多类似setCookie()的函数，name与value必填，但其他的都不是。既然其他数据没有优先顺序，就有道理使用带命名属性的可选对象替代额外的命名参数。这个方法很好，尽管它使得期待的输入对函数有一点不透明。

使用解构参数，之前的可以被重写如下：
```javascriipt
function setCookie(name, value, { secure, path, domain, expires }) {

    // ...}

setCookie("type", "js", {
    secure: true,
    expires: 60000
});
```
这个函数的行为与之前的例子相似，最大的不同点是第三个参数使用解构取出了必要的数据。这样做哪些参数是真的需要就一清二楚了，而在xxx中解构参数也表现得像常规的参数一样，如果它们被设置为undefined就不会被传递。

这种模式的其中一种怪癖是，如果解构参数为提供时将抛出错误。如果setCookie()调用时只带两个参数，它返回一个允许时错误。
```javascript
// Error!
setCookie("type", "js");
```
这段代码抛出错误，因为第三个参数缺失（undefined）。要理解为什么这是一个错误，它有助于理解解构参数它真的只是一个解构赋值的速记而已。JavaScript引擎实际上是这样做的：
```javascript
function setCookie(name, value, options) {

    var { secure, path, domain, expires } = options;

    // ...
}
```
因为解构赋值在右边的表达式被解释为null或undefined时抛出错误，第三个参数未传时同样如此。

你可以通过为解构参数传递一个默认值来解决这一行为:
```javascript
function setCookie(name, value, { secure, path, domain, expires } = {}) {

    // ...
}
```
这样例子现在同本节中的第一个例子完全一样了，为解构参数提供默认值意味着如果setCookie()的第三个参数未提供的话，secure, path, domain和 expires都是undefined。

注意：推荐总是为解构参数提供默认值以避免所有使用时的独特错误。

#### 展开运算符

展开运算符与多余参数密切相关。剩余运算符允许你指定多个独立的可以组合成一个数组的参数，而扩展运算符允许你指定一个可以被分割的每一项可以像独立参数一样传给函数的数组。想想Math.Max()方法，它接收任何数量的参数并返回其中一个最高值。它的基本用法如下：
```javascript
let value1 = 25,
    value2 = 50;
console.log(Math.max(value1, value2));      // 50
```
当你只是处理两个值时，正如这个例子中，Math.Max()很容易使用。传入两个值，然后返回更高一点的值。但如果是在数值中跟踪值呢？现在你想要找最高的值？Math.Max()方法不允许你传入数组，所以在ECMAScript5及更早，你会坚持自己搜索数组或者使用apply()：
```javascript
let values = [25, 50, 75, 100]

console.log(Math.max.apply(Math, values));  // 100
```
而可能的是，以这种方式使用apply()有一点令人迷惑——带上额外的语法实际上它似乎混淆了代码的真实含义。

ECMAScript6开展运算符使这种情况非常简单。你可以传入一个数组，用剩余参数中同样使用的...模式作为前缀来代替调用apply()。JavaScript引擎就会将数组分割成个体而后传入：
```javascript
let values = [25, 50, 75, 100]

// equivalent to
// console.log(Math.max(25, 50, 75, 100));
console.log(Math.max(...values));           // 100
```
现在调用Math.Max()看起来更方便点了，且避免了简单数学操作中的this绑定的复杂性。

你同样可以用其它参数混合和匹配展开运算符，假设你想要Math.max()的最大值返回0（负数潜入数组的情况）你可以单独传入这个参数，然后对其它参数还是使用展开运算符：
```javascript
let values = [-25, -50, -75, -100]

console.log(Math.max(...values, 0));        // 0
```
此例中，最后一个传入Math.max()的参数是0，它在用展开运算符传入的其它参数后面。

参数传递的展开运算符使得对函数参数使用数组更加简单。你可能会发现在大多数情况下，它可合理取代apply()方法。

#### name属性

在JavaScript中识别函数是很有挑战性的，函数定义有多种方式。此外，匿名函数的流行使得调试变得更加困难，经常导致堆栈跟踪难以阅读和解释。由于这些原因，ECMAScript对所有函数添加了name属性。在ECMAScript6程序的所有函数的name属性都会有一个适当值，而其它的将会是空字符串。比如：
```javascript
function doSomething() {
    // ...}

var doAnotherThing = function() {
    // ...};

console.log(doSomething.name);          // "doSomething"
console.log(doAnotherThing.name);       // "doAnotherThing"
```
这段代码中，doSomething()有一个等于"doSomething"的name属性，因为它是一个函数声明。匿名函数表达式doAnotherThing()有一个"doAnotherThing"的name是由于它所指派的变量。

上个例子中虽然函数声明和函数表达式都很容易为之找到一个合适的name，而ECMAScript进一步保证了所有的函数有一个合适的name：
```javascript
var doSomething = function doSomethingElse() {
    // ...};

var person = {
    get firstName() {
        return "Nicholas"
    },
    sayName: function() {
        console.log(this.name);
    }
}

console.log(doSomething.name);      // "doSomethingElse"
console.log(person.sayName.name);   // "sayName"
console.log(person.firstName.name); // "get firstName"
```
此例中， 

doSomething.name是 "doSomethingElse"因为函数表达式本身也有一个name，它优先于函数指派的变量。 person.sayName()的name属性是"sayName",因为值解释来自对象字面量。相似的，personfirstName事实上是一个getter方法，所以为表明这种差异，它的name是"get firstName"（同样地，setter方法用"set"作为前置）。关于函数名有几个其他的特殊情况。使用bind()创建的函数它们的name用"bound"作为前缀，而用Function构造器创建的函数name为"anonymous"。
```javascript
var doSomething = function() {
    // ...};

console.log(doSomething.bind().name);   // "bound doSomething"
console.log((new Function()).name);     // "anonymous"
```
绑定函数的name总是用被绑定函数名前缀"bound"，所以doSomething()的绑定版本是"bound doSomething"。

#### 块级函数

在ECMAScript3及更早，出现在块（块级函数）内的函数声明是一个技术上的语法错误，但是许多浏览器仍旧支持。不幸地是，每个浏览器允许的语法表现的方式都略有不同。因此，避免块内函数声明被认为是最佳实践（最好的选择是使用函数表达式）。

为解决这种不兼容的行为，ECMAScript5严格模式引入了函数声明在块中被使用时的错误，比如：
```javascript
"use strict";

if (true) {

    // Throws a syntax error in ES5, not so in ES6function doSomething() {
        // ...    }
}
```
ECMAScript5中，这段代码抛出一个语法错误，在ECMAScript6中，doSomething()方法被认为是一个块级声明，它可以在被定义的同一个块内被访问和调用，比如：
```javascript
"use strict";

if (true) {

    console.log(typeof doSomething);        // "function"function doSomething() {
        // ...    }

    doSomething();
}

console.log(typeof doSomething);            // "undefined"
```
块级函数被提升到被定义的块的顶部，所以

typeof doSomething 返回“function”尽管代码中它出现在函数声明的前面。一旦if块执行完成，doSomething()不再存在。

一旦执行流离开函数定义所在的块，函数定义就被移除，在这一点上块级函数能类似let函数表达式。关键的不同点是块级函数被提升到包含块的顶部，而let函数表达式不被提升。比如：
```javascript
"use strict";

if (true) {

    console.log(typeof doSomething);        // throws error
    let doSomething = function () {
        // ...    }

    doSomething();
}

console.log(typeof doSomething);
```
这里，typeof doSomething执行完后代码就结束执行，因为let语句还未被执行。

你要使用块级函数还是let表达式取决于你是否想要提升行为。

ECMAScript6也允许在非严格模式下使用块级函数，但行为略有不同。这些声明不是被提升到块顶部，而是在包含块和全局环境中都被提升。比如：
```javascript
// ECMAScript 6 behaviorif (true) {

    console.log(typeof doSomething);        // "function"function doSomething() {
        // ...    }

    doSomething();
}

console.log(typeof doSomething);            // "function"
```
此例中，doSomething()被提升到全局作用域，所以在块外面依旧存在。ECMAScript6标准化了这一行为以移除之前存在的不兼容的浏览器行为。ECMAScript6运行时就会全部表现一致。

#### 箭头函数

ECMAScript中最有意思的部分之一是箭头函数。顾名思义，箭头函数是使用“箭头”(=>)新语法定义的函数。然而，箭头函数与传统JavaScript函数在一些重要的方面表现不同。

词汇this绑定——函数内this的值由箭头函数在哪里定义决定，而不是由它在哪里使用决定。

不可再生——箭头函数不可被作为构造器使用，如果使用时带new将抛出异常。

this不可改变——函数内的this不可改变，它在函数的整个生命周期中保持相同的值。

没有arguments对象——你不能通过arguments对象访问参数，你必须使用命名参数或者其他ES6特征比如剩余参数。

这些差异存在有几个原因。首先，this绑定在JavaScript中是一个常见的错误来源，很容易丢失函数内对这个值的跟踪将招致意外的后果。第二，通过限制箭头函数简单地执行带单个this值的代码，JavaScript可以更容易优化这些操作（与可能会被用来做构造函数或其他修改的常规函数相对）。

注意：箭头函数与其他函数遵循同样的规则，它也有name属性。

##### **语法**

箭头函数的语法有很多种，它取决于你试图完成什么。所有的变化从函数参数开始，后面跟箭头，再后面跟函数体。所有的参数和主体可以根据用法采用不同的形式。比如，下面的箭头函数传入一个参数然后简单得返回它：
```javascript
var reflect = value => value;

// effectively equivalent to:var reflect = function(value) {
    return value;
};
```
如果箭头函数中只有一个参数，这一个参数可以不需要更多的语法直接使用，接下来的箭头和箭头右边的表达式被求值并返回。尽管那里没有明确的返回语句，箭头函数会返回传入的第一个参数。

如果你传入不只一个参数，你必须用括号将这些参数扩起来，比如：
```javascript
var sum = (num1, num2) => num1 + num2;

// effectively equivalent to:var sum = function(num1, num2) {
    return num1 + num2;
};
```
sum()函数简单地相加两个参数并返回结果。唯一的不同是参数用小括号括起来，参数之间用逗号分隔（正如传统的函数一样）。如果函数没有参数，你必须包括一对空括号。
```javascript
var getName = () => "Nicholas";

// effectively equivalent to:var getName = function() {
    return "Nicholas";
};
```
当你想提供一个更加传统的函数体，也许它由一个以上的表达式组成，这时你需要用大括号将函数体括起来，然后显示地定义一个返回值，比如：
```javascript
var sum = (num1, num2) => {
    return num1 + num2;
};

// effectively equivalent to:var sum = function(num1, num2) {
    return num1 + num2;
};
```
你可以或多或少地将花括号内的参数与传统函数中的同等对待，除了参数不可用的例外。

如果你想要创建一个什么都不做的函数，你就需要包含花括号：
```javascript
var doNothing = () => {};

// effectively equivalent to:var doNothing = function() {};
```
因为花括号用来表示函数体，想要从函数体外返回对象字面量的箭头函数必须用圆括号包裹字面量。比如：
```javascript
var getTempItem = id => ({ id: id, name: "Temp" });

// effectively equivalent to:var getTempItem = function(id) {

    return {
        id: id,
        name: "Temp"
    };
};
```
用圆括号包裹对象字面量表明圆括号是一个对象字面量而不是函数体。

##### **立即调用函数表达式（IIFEs）**

JavaScript中函数的一个流行应用是立即调用函数表达式（IIFEs）。IIFEs允许你定义一个匿名函数，然后立即调用它而不保存引用。当你想要创建一个对剩余程序屏蔽的域，这种模式就派上用场了。比如：
```javascript
let person = (function(name) {

    return {
        getName() {
            return name;
        }
    };

}("Nicholas"));

console.log(person.getName());      // "Nicholas"
```
这段代码中，IIFE被用来创建一个带有getName()方法的对象，这个方法用name参数做返回值，有效地使name成为了被返回对象的私有成员。

你可以用箭头函数来完成同样的事，只要你用小括号将箭头函数括起来：
```javascript
let person = ((name) => {

    return {
        getName() {
            return name;
        }
    };

})("Nicholas");

console.log(person.getName());      // "Nicholas"
```
注意这个例子中括号的位置与前面的不同。前面的例子使用函数关键字用小括号将整个表达式包裹起来，包括传给函数的参数“Nicholas”，这个例子只有箭头函数外面有小括号，然后传入参数。

##### **词法this绑定**

JavaScript中最常见的误区之一就是函数内的this绑定。因为单一函数中this的值取决于它被调用时的上下文，是可变的。当你想要影响一个对象时很有可能错误地影响另一个对象。考虑下面的例子：
```javascript
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click", function(event) {
            this.doSomething(event.type);     // error
        }, false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```
这段代码中，对象

PageHandler 目的在于处理页面上的交互。init()方法被调用以建立交互，而后这个方法反过来指定一个事件处理程序来调用this.doSomething()。然而，这段代码并不能按预期工作。对this.doSomething()的调用中断了因为this是对元素对象（在这里的情况是document）的引用，是事件的目标，而不是绑定到PageHandler。如果你试着运行这段代码，事件处理程序激活时你会得到一个错误，因为this.doSomething()在目标document对象中不存在。

你可以在函数中明确使用bind()方法将this的值绑定到PageHandler ：
```javascript
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click", (function(event) {
            this.doSomething(event.type);     // no error
        }).bind(this), false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```
现在，代码能如预期工作，但是可能会看起来有点奇怪。通过调用bind(this)，你确实创建了一个新的、this被绑定到当前this即PageHandler的函数。现在这段代码按你期待的方式工作，尽管你不得不创建一个额外的函数以完成这项工作。

箭头函数内含了this绑定。这意味着箭头函数中this的值总是同箭头函数定义时的域中this的值一致。比如：
```javascript
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click",
                event => this.doSomething(event.type), false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```
这个例子中事件处理程序是一个调用了this.doSomething()的箭头函数。this的值同没有init()时是一样的，所以这个版本的例子与使用bind()的那个相比要更加简单。尽管doSomething()方法不返回值，对函数体而言它仍然是有必要执行的唯一语句，所以没有必要包含大括号。

箭头函数的目的是成为一次性函数，所以不能用来定义新的类型。显而易见，这是丢失了常规函数有的原型属性。如果你尝试对箭头函数使用new操作符，你会得到一个错误：
```javascript
var MyType = () => {},
    object = new MyType();  // error - you can't use arrow functions with 'new'
```
既然，this的值是静态绑定到this函数的，你不能用call()、apply()或者bind()来改变this的值。

箭头函数的简明语法使得用它们做数组处理成为了最佳选择。比如，如果你想要用自定义比较来排序数组，你通常会这样写：
```javascript
var result = values.sort(function(a, b) {
    return a - b;
});
```
对一个非常简单的程序而言，这里的语法就过多了。相比之下，更简洁的箭头函数版本：
```javascript
var result = values.sort((a, b) => a - b);
```
接受了回调函数比如sort(), map()和 reduce()的箭头方法都可以受益于带箭头函数的简单语法，它将看起来复杂的程序转化为简单的代码。

一般来说，箭头函数被设计用在匿名函数通常被使用的地方，它们的目的真的不是用来长时间保持，因此无法将箭头函数作为构造器使用。箭头函数用于传给其他函数的回调是最好的，正如我们在这节中看到的例子一样。

##### **词法arguments绑定**

尽管箭头函数没有它们自己的arguments对象，它们也有可能接受一个来自包含函数的的arguments对象。那个arguments对象是可用的，不管后来箭头函数在哪里执行。比如：
```javascript
function createArrowFunctionReturningFirstArg() {
    return () => arguments[0];
}

var arrowFunction = createArrowFunctionReturningFirstArg(5);

console.log(arrowFunction());       // 5
```
在createArrowFunctionReturningFirstArg()中，arguments[0]被创建的箭头函数引用。这个引用包含了传给createArrowFunctionReturningFirstArg()的第一个参数，当箭头函数后来执行的时候，它返回5，这就是传给createArrowFunctionReturningFirstArg()的第一个参数。尽管箭头函数不再处于创建它的函数的域中，arguments作为一个词法绑定保持可用。

##### **标识箭头函数**

尽管有不同的语法，箭头函数仍然是函数，它们这样被识别：
```javascript
var comparator = (a, b) => a - b;

console.log(typeof comparator);                 // "function"
console.log(comparator instanceof Function);    // true
```
typeof 和instanceof 在处理箭头函数时同其他函数都表现一致。

也和其他函数一样，你还是可以使用call(),apply()和bind()尽管这个函数的this绑定并不会被影响。下面是一些例子：
```javascript
var sum = (num1, num2) => num1 + num2;

console.log(sum.call(null, 1, 2));      // 3
console.log(sum.apply(null, [1, 2]));   // 3var boundSum = sum.bind(null, 1, 2);

console.log(boundSum());                // 3
```
这个例子中，用call()和apply()调用sum()函数，像你在任何函数中一样传入参数。bind()方法用于创建boundSum()，他自己有两个参数绑定到1和2，所以它们不需要被直接传递。

箭头函数适合用在任何你当前在使用匿名函数表达式的地方，比如回调。

#### 总结

在ECMAScript6中，函数并未经历巨大改变。但有相当一系列的增量变化使得它们的使用更简单。

当一个特定的参数没有指定时，默认函数参数允许你简单的指定想要使用的值。在ECMAScript6之前，这将需要一些额外的代码在函数中检测参数是否存在并分配不同的值。

剩余函数允许你简单的将所有应该被放置的剩余参数指定为一个数组，使用一个新数组，然后让你指出要包含哪些参数使得剩余参数成为比arguments更加灵活的解决方案。

解构参数使用解构语法使得使用函数参数时选项对象更加透明。你感兴趣的实际数据可以与其他命名参数一起列出来。

展开运算符是剩余参数的同伴，允许你在调用函数时将数组解构成单独的参数。在ECMAScript6之前，传递包含在数组中的单个参数的唯一方式是要么手动指定每个参数要么使用apply()。有了展开运算符，你可以简单地对任何函数传递数组而不必担心函数的this绑定。

name属性的添加有助于调试和执行时更容易识别函数。此外，ECMAScript6正式地定义了块级函数行为，因此严格模式下它们不再是一个语法错误。

在ECMAScript6中对函数而言最大的改变是箭头函数的添加。箭头函数是设计使用在匿名函数通常使用的地方。箭头函数有更加简洁的语法，词法this绑定，且没有arguments对象。此外，箭头函数不能改变它们的this绑定所以不能被作为构造器使用。