title: '[译]Understanding ECMAScript6 基本知识'
tags:
  - Understanding ECMAScript 6
categories: []
date: 2015-11-23 18:43:00
---
### 基本知识

ECMAScript 6在ECMAScript 5之上做了大量的改变。一些改变很大，比如添加新的类型或者语法，而其它的非常小，提供了语言之上的渐进改进。这个章节包含了那些渐进改进，它们可能不会获得很多关注但提供了一些重要的功能，使得某些类型的问题更容易解决。

#### **更好的Unicode支持**

ECMAScript 6之前， JavaScript是完全基于16位字符编码的想法。所有的字符串属性和方法，比如length与charAt() ，是基于每一个16位序列表示一个字符这一想法。ECMAScript 5 允许JavaScript 引擎来决定使用UCS-2 或者UTF-16 (两个编码都使用16位编码单元，所有的操作是一样的)这两个编码中的哪一个。确实世界上所有的字符串曾一度适合16位，然而情况已不再如此。就Unicode对世界上的每一个字符提供全球唯一标识的既定目标而言，保持16位是不可能的。这些全球唯一标识，被称为码点，只是简单的从0（你可能会认为这些是字符编码，但是有些微差异）开始的数字。字符编码负责将码点编码成内部一致的编码单元 。UCS-2 有一对一的映射，码点到编码单元，UTF-16则更加灵活。

在 UTF-16中，第一个 2^16 码点被表示为单个16位编码单元。称为基本多文种平面Basic Multilingual Plane (BMP)。一切超出这个范围的东西被认为是辅助平面，这里的码点不能再被表示为仅仅16位。UTF-16通过引入代理对解决了这个问题，在这里，一个单一的码点被表示为两个16位编码单元。这意味着字符串中的任意一个单一字符可以是一个编码单元（对于MBP,总共16位）或者两个编码单元 (对于复制平面字符，总共16位)。

ECMAScript 5保持所有的操作都工作在16位编码单元，这意味着你可以从包含了代理对的字符串中得到意想不到的结果。比如：
```javascript
var text = "𠮷";

console.log(text.length);           // 2
console.log(/^.$/.test(text));      // false
console.log(text.charAt(0));        // ""
console.log(text.charAt(1));        // ""
console.log(text.charCodeAt(0));    // 55362
console.log(text.charCodeAt(1));    // 57271
```
在这个例子中，一个单一的Unicode字符用代理对来表示，因此，JavaScript字符串操作将这个字符串作为两个16位的字符处理。这意味着长度为2，常规的正则表达式尝试匹配单一字符失败，charAt()也不能返回一个有效的字符串。 charCodeAt()方法为每一编码单元返回对应的16位数字，这是你在ECMAScript5中能得到的最接近真实值的东西。

ECMAScript 6在UTF-16中执行字符串编码。字符编码的规范意味着这个语言现在可以支持功能性的设计来专门处理代理对。

##### **codePointAt() 方法**

完全支持UTF-16方法的第一个例子是codePointAt(),它可以用来检索字符串中映射到给定位置的Unicode码点。这个方法接收编码单元的位置（不是字符位置）作为参数然后返回一个整型值。

```javascript
var text = "𠮷a";

console.log(text.charCodeAt(0));    // 55362
console.log(text.charCodeAt(1));    // 57271
console.log(text.charCodeAt(2));    // 97
console.log(text.codePointAt(0));   // 134071
console.log(text.codePointAt(1));   // 57271
console.log(text.codePointAt(2));   // 97
```

除了非BMP字符，codePointAt()方法与charCodeAt()方法一样的结果是一样的。文本中的第一个字符是非BMP字符因此它由两个编码单元组成，这意味着整个字符串的长度是3而不是2。charCodeAt()方法仅仅返回位置0对应的第一个编码单元，而codePointAt()返回完整的码点尽管它跨越了多个编码单元。对于位置1和2，这两个方法返回相同的值（第一个字符的第二个编码单元)和(“a”)。

这个方法是判断一个给定的字符由一个编码单元组成还是两个编码单元组成的最简单的方式：
```javascript
function is32Bit(c) {
    return c.codePointAt(0) &gt; 0xFFFF;
}

console.log(is32Bit("𠮷"));         // true
console.log(is32Bit("a"));          // false
```
16位字符的上限由16进制FFFF表示，所以任何码点在这个数字之上的字符必定被表示为两个编码单元。

##### **String.fromCodePoint()**

当ECMAScript 提供了一个方法来做某件事, 它还会提供一个方法来做相反的事。你可以在字符串中使用codePointAt()来检索某个字符的码点，而String.fromCodePoint()为给定的码点提供一个单一字符字符串。比如：

console.log(String.fromCodePoint(134071));  // "𠮷"

你可以把String.fromCodePoint认为是String.fromCharCode的更加完善的版本。对BMP字符而言，这两个方法返回同样的结果。唯一的不同便是当字符超出BMP范围的时候。

##### **转义非BMP字符**

ECMAScript5允许字符串包含由转义序列来表示的16位Unicode字符。转义序列是\u后面跟4位16进制值。比如，转义序列\u0061表示字母“a”:
```javascript
console.log("\u0061");      // "a"
```
如果你试图使用数值超过FFFF（即BMP上限）的转义序列，你将得到一些令人吃惊的结果：
```javascript
console.log("\u20BB7");     // "7"
```
因为Unicode转义序列被定义为总是有精确的四位16进制字符，ECMAScript将\u20bb7解析为两个字符： \u20BB 和 "7" 第一个字符不可打印，第二个就是数值7。

ECMAScript6通过引入扩展的Unicode转义序列解决了这个问题，将16进制数字用一对花括号括起来。这使得我们可以用任何数量的十六进制字符来制定一个单一字符：
```javascript
console.log("\u{20BB7}");     // "𠮷"
```
使用扩展的转义序列，正确的字符会被包含在字符串中。

警告：确保你只在ECMAScript6环境中使用这个新的转义序列。在所有其他环境中，这样做会导致语法错误。你可以通过如下方法来检测和查看当前环境是否支持扩展转义序列：
```javascript
function supportsExtendedEscape() {
    try {
        eval("'\\u{00FF1}'");
        returntrue;
    } catch (ex) {
        returnfalse;
    }
}
```
##### **normalize() 方法**

Unicode另一有趣的方面是不同的字符可能会因为排序或者其他基于比较的操作被视为等价。有两种方式来定义这些关系。第一，正则等价，意思是说两个序列的码点在各方面被认为是可互换的。这甚至意味着两个字符的结合对另一个字符而言可以是正则等价的。第二个关系是兼容性，意思是说两个序列的码点有不同的外观，但是在某些情况下可以互换使用。

重要的是了解，由于这些关系，两个字符串从本质上表示相同的文本但是有着各自不同码点序列的情况是可能的。举个例子，字符“æ”和字符串“ae”可以互换使用尽管它们有着不同的码点。在JavaScript中，这两个字符串不相等除非它们以某种方式规范化。

ECMAScript6通过字符串的新方法normalize()支持四位Unicode的标准化规范格式。这个方法选择性地接受一个单一的参数，"NFC" (默认), "NFD", "NFKC"或"NFKD"中的一个。解释这四种形式的区别超出了本书范围。 只是要记住，为了使用，你必须将这两个字符串以相同的形式进行标准化。比如：
```javascript
var normalized = values.map(function(text) {
    return text.normalize();
});

normalized.sort(function(first, second) {
    if (first &lt; second) {
        return -1;
    } elseif (first === second) {
        return 0;
    } else {
        return 1;
    }
});
```
在这段代码中，在values数组中的字符串被转为标准化的格式，因此数组可以被适当地排序。你可以通过调用normalize()作为比较器的一部分在原始数组上完成排序：
```javascript
values.sort(function(first, second) {
    var firstNormalized = first.normalize(),
        secondNormalized = second.normalize();

    if (firstNormalized &lt; secondNormalized) {
        return -1;
    } elseif (firstNormalized === secondNormalized) {
        return 0;
    } else {
        return 1;
    }
});
```
再次强调，要记住的最重要的事情是两个值必须以同样的方式标准化。下面的例子使用了默认的NFC，但是你可以简单地指定其它中的一个：

如果你之前从未担心过Unicode标准化，可能你使用这个方法的时候不会有很多。然而，知道它是可用的会有助于你在国际化的应用中工作更好。

##### **正则表达式u标识**

许多常见的字符串操作是通过使用正则表达式完成的。然而，正如前面所提到的，正则表达式的工作也是基于16位编码单元，每一个代表一个单一的字符串。这就是为什么单一的字符匹配在之前的例子中无效的原因。为解决这一问题，ECMAScript为正则表达式定义了一个新的标志："Unicode"的u。

当一个正则表达式有u标志，它将工作模式切换到字符而不是编码单元。这意味着正则表达式将不用再对字符串中的代理对感到困惑，它可以像预期那样表现。比如：
```javascript
var text = "𠮷";

console.log(text.length);           // 2
console.log(/^.$/.test(text));      // false
console.log(/^.$/u.test(text));     // true
```
添加u标志允许正则表达式按照字符正确匹配字符串。不幸的是，ECMAScript还没有一种方法来确定一个字符串中有多少码点。幸运的是，我们可以用正则表达式计算出来：
```javascript
function codePointLength(text) {
    var result = text.match(/[\s\S]/gu);
    return result ? result.length : 0;
}

console.log(codePointLength("abc"));    // 3
console.log(codePointLength("𠮷bc"));   // 3
```
在这个例子中，正则表达式同时匹配了空白和非空白字符，因为Unicode的启用，它在全球范围内应用。result中包含匹配结果的数组，至少有一个匹配，所以数组的长度最终是字符串中码点的数量。

警告：尽管这种方法有效，但它不是非常快，尤其是运用在长字符串时。如果可能的话，尽量减少码点计算。希望ECMAScript7会带来一种更高性能的计算码点的方式。

由于u标识是一个语法变化，试图在不一致的JavaScript引擎中使用它意味着抛出语法错误，用一个函数来确定是否支持u标识是最安全的方式：
```javascript
function hasRegExpU() {
    try {
        var pattern = new RegExp(".", "u");
        returntrue;
    } catch (ex) {
        returnfalse;
    }
}
```
此方法使用Repexp构造器将u标识作为参数传入，即便在更老的javascript引擎中这也是有效的语法，然而，如果u不被支持，构造器将抛出一个错误。

注意：如果你的代码依然需要在更老的javascript引擎中工作，使用u标识时最好只使用Regexp构造器。这样可以防止语法错误，允许你选择性地检测和使用u标识而不中止执行。

#### 其他字符串变化

JavaScript字符串总是落后于其他语言的类似功能。 只是在ECMAScript5中最后增加了一个trim()方法，而ECMAScript6继续为字符串扩展了新功能。

##### **includes(), startsWith(), endsWith()**

自从JavaScript被首次引入，开发者们用indexOf()方法从其他字符串中查找字符串。ECMAScript6添加了3个新方法用于从其他字符串中识别字符串：

- includes() - 如果给定的文本从字符串的任何位置被找到就返回真，否则返回假。
- startsWith() - 如果给定的文本在字符串的起始位置被找到就返回真，否则返回假。
- endsWith() - 如果给定的文本在字符串的结束位置被找到就返回真，否则返回假。

这些方法每一个都接收两个参数：要查找的文本和可选的从何处开始搜索的位置。省略第二个参数时，includes()和startsWith()从字符串起始位置开始搜索而endsWith()从字符串的结束位置开始搜索。实际上，第二个参数减少了被搜索的字符串。下面是一些例子：
```javascript
var msg = "Hello world!";

console.log(msg.startsWith("Hello"));       // true
console.log(msg.endsWith("!"));             // true
console.log(msg.includes("o"));             // true
console.log(msg.startsWith("o"));           // false
console.log(msg.endsWith("world!"));        // true
console.log(msg.includes("x"));             // false
console.log(msg.startsWith("o", 4));        // true
console.log(msg.endsWith("o", 8));          // true
console.log(msg.includes("o", 8));          // false
```
这三个方法使识别子字符串更加容易，不需要再担心识别它们的确切位置。

注意：这些方法都返回一个布尔值。如果你想要找到字符串在另一字符串中的位置，使用indexOf()或lastIndexOf()。

警告：如果你传了一个正则表达式而不是字符串，startsWith(),endsWith()和includes()方法将抛出一个错误。这与indexOf()方法和lastIndexOf()方法形成了鲜明对比，它们都会将正则表达式转为字符串然后再查找字符串。

##### **repeat()**

ECMAScript 6也给字符串增加了repeat()方法， 这个方法只接收一个参数，即重复字符串的次数，返回一个新的对原始字符串重复了指定次数的字符串。比如：
```javascript
console.log("x".repeat(3));         // "xxx"
console.log("hello".repeat(2));     // "hellohello"
console.log("abc".repeat(4));       // "abcabcabcabc"
```
这个方法是一个真正的高于一切的便利的功能，当处理文本操作时会特别有用。举一个例子，当你想要用代码格式化工具创建缩进级别时，这个功能就非常有用：
```javascript
// indent using a specified number of spacesvar indent = " ".repeat(size),
    indentLevel = 0;

// whenever you increase the indentvar newIndent = indent.repeat(++indentLevel);
```
#### 其他正则表达式变化

在JAVAScript中，正则表达式是处理字符串时的一个重要组成部分，像这个语言的其他部分一样，在最近版本中真的没有太多改变。然而，ECMAScript6对正则表达式提出了一些改进以支持字符串的更新。

##### **正则表达式y标识**

ECMAScript 6标准化了y标识，自从它作为正则表达式的一个专有扩展在Firefox中被实现后。  y (粘滞) 标识表示下一个匹配应该从正则表达式lastIndex的值开始。

 lastIndex 属性表示从字符串的哪个位置开始匹配，默认被置为0。意思是匹配总是从字符串的起始位置开始。然而，你可以重写lastIndex使其从别的地方开始：
```javascript
var pattern = /hello\d\s?/g,
    text = "hello1 hello2 hello3",
    result = pattern.exec(text);

console.log(result[0]);     // "hello1 "
pattern.lastIndex = 7;
result = pattern.exec(text);

console.log(result[0]);     // "hello2 "
```
在这个例子中，正则表达式匹配“hello”后面跟一个数字或者可选的空白符的字符串。g标识非常重要因为如果设置了此标识的话它允许正则表达式使用lastIndex(没有它，匹配总是，无视lastIndex的值)。第一次调用exec()方法时结果是先匹配“hello1”，而第二次调用时，因为lastIndex为7，首先匹配“hello2”。

粘滞标识告诉正则表达式在最近一次匹配后保存下一个字符的索引到lastIndex,无论操作是否完成（在前面的例子中，7是“hello1”之后下一个字符的位置）。如果一个操作的结果是没有匹配则lastIndex重置为0。
```javascript
var pattern = /hello\d\s?/y,
    text = "hello1 hello2 hello3",
    result = pattern.exec(text);

console.log(result[0]);             // "hello1 "
console.log(pattern.lastIndex);     // 7
result = pattern.exec(text);

console.log(result[0]);             // "hello2 "
console.log(pattern.lastIndex);     // 14
```
这里，使用了相同的模式但是用粘滞标识代替了全局标识。第一次调用exec()之后，lastIndex的值变为7，在第二次调用之后变为14。因为粘滞标识会为你更新lastIndex,你自己就没有必要跟踪然后手动更新了。 

对于粘滞标识，也许最重要的事情是理解粘滞标识正则表达式在起始位置有一个暗指的 ^ ， 表示模式要从数据的起始位置开始匹配。比如：如果前面的例子改为不匹配空白符，结果就不同了：
```javascript
var pattern = /hello\d/y,
    text = "hello1 hello2 hello3",
    result = pattern.exec(text);

console.log(result[0]);             // "hello1"
console.log(pattern.lastIndex);     // 6
result = pattern.exec(text);

console.log(result);                // null
console.log(pattern.lastIndex);     // 0
```
不匹配空白符时，第一次调用exec（）后lastIndex被设为6。这意味着正则表达式会这样解析字符串：

" hello2 hello3"

因为在正则表达式模式起始位置有一个隐性的^，模式从“h”开始匹配，（开头）不是空白符，发现它们不等价。匹配从那里就结束了，然后返回null。lastIndex的值被重置为0。

至于其他正则表达式标识，你可以检测通过使用一个属性来检测y是否存在。如果粘滞标识存在，sticky属性被设为true，如果不存在则被设为false。
```javascript
var pattern = /hello\d/y;

console.log(pattern.sticky);    // true
```
sticky是基于粘滞标识存在的只读属性，因而不能在代码中更改。

注意：lastIndex属性仅当调用正则表达式对象的方法如exec()和test()时有效。将正则表达式传给一个字符串方法比如match()，不会引起粘滞行为。

同u标识相似，y标识也是一个语法变化，所以在老版本的JavaScript引擎中它也会导致语法错误。你可以使用同样的方法来检测支持：
```javascript
function hasRegExpY() {
    try {
        var pattern = new RegExp(".", "y");
        returntrue;
    } catch (ex) {
        returnfalse;
    }
}
```
同样与u相似，如果你需要再老的JavaScript引擎中使用u，当定义这些正则表达式时确保使用RegExp构造器以避免语法错误。

##### **复制正则表达式**

在ECMAScript 5, 你可以通过将正则表达式传给RegExp构造器来复制正则表达式，比如：
```javascript
var re1 = /ab/i,
    re2 = new RegExp(re1);
```
然而，如果你给RegExp传了第二个参数，为正则表达式指定了标识的话，就会抛出错误：
```javascript
var re1 = /ab/i,

    // throws an error in ES5, okay in ES6
    re2 = new RegExp(re1, "g");
```
如果你在ECMAScript5环境中执行这段代码，你会得到一个声明第一个参数是正则表达式时第二个参数不能被使用的错误。 ECMAScript 6改变了这一行为，这样第二个参数是允许的，它会覆盖第一个参数上设置的标识。比如：
```javascript
var re1 = /ab/i,

    // throws an error in ES5, okay in ES6
    re2 = new RegExp(re1, "g");

console.log(re1.toString());            // "/ab/i"
console.log(re2.toString());            // "/ab/g"
console.log(re1.test("ab"));            // true
console.log(re2.test("ab"));            // true
console.log(re1.test("AB"));            // true
console.log(re2.test("AB"));            // false
```
在这段代码中，re1设置了不区分大小写的i标识，而re2只有全局g标识。RegExp构造器从re1中复制了模式，然后用g替代i。如果没有第二个参数，re2和re1会有相同的标识。

##### **flags属性**

在ECMAScript 5, 通过source属性获取正则表达式的文本是可能的，但是要得到标识字符串需要解析toString()输出，比如：
```javascript
function getFlags(re) {
    var text = re.toString();
    return text.substring(text.lastIndexOf("/") + 1, text.length);
}

// toString() is "/ab/g"var re = /ab/g;

console.log(getFlags(re));          // "g"
```
ECMAScript 6增加了一个flags属性来配合source。两个属性都是原型访问器属性，只指定了一个getter（使它们只读）。flags的添加使调试和继承为目的的正则表达式检查更加容易。

ECMAScript 6后期添加的flags属性，返回所有应用在正则表达式的标识的字符串表示。比如：
```javascript
var re = /ab/g;

console.log(re.source);     // "ab"
console.log(re.flags);      // "g"
```
source和flags一起用是很有必要的，允许你提取正则表达式片段，不再需要直接将正则表达式转为字符串。

#### Object.is()

当你要比较两个值，可能你已习惯于使用等于操作符（==）或者全等操作符（===）。许多人更喜欢后者以避免比较时的类型强制转换。然而，即便是全等操作符也不完全准确。比如，值 +0 和 -0 在 === 比较时被认为是相等的，尽管他们在JavaScript引擎中表现不同。NaN === NaN 也返回false, 它需要使用isNaN()来正确检测NaN 。

ECMAScript 6引入了Object.is() 来弥补全等操作符遗留下来的怪癖 。这个方法接收两个参数，如果值是相等的就返回true。两个值如果类型相同且值相同就被认为是相等的。很多情况下，Object.is()与===的效果是一样的，唯一的不同就是+0和-0被认为不相等，而NaN被认为与NaN是相等的。下面是一些例子：
```javascript
console.log(+0 == -0);              // true
console.log(+0 === -0);             // true
console.log(Object.is(+0, -0));     // false
console.log(NaN == NaN);            // false
console.log(NaN === NaN);           // false
console.log(Object.is(NaN, NaN));   // true
console.log(5 == 5);                // true
console.log(5 == "5");              // true
console.log(5 === 5);               // true
console.log(5 === "5");             // false
console.log(Object.is(5, 5));       // true
console.log(Object.is(5, "5"));     // false
```
大多数情况下你大概还是想使用==或者===来比较值，Object.is()作为特殊的情况可能不会直接影响到你。

#### 块绑定

传统上，javascript其中一个棘手的部分就是var声明。在大多数基于C的语言中，变量在声明的地方被创建。然而，在javascript中，情况并非如此。使用var声明的变量被提升到函数顶部（或者全局作用域），不管实际的声明发生在什么地方。比如：
```javascript
function getValue(condition) {

    if (condition) {
        var value = "blue";

        // other codereturn value;
    } else {

        // value exists here with a value of undefinedreturnnull;
    }

    // value exists here with a value of undefined
}
```
如果你对JavaScript不熟悉，或许会认为变量value只有在condition解析为true时被定义，事实上，变量value已经不管不顾地被声明了。JavaScript引擎将代码改成这般：
```javascript
function getValue(condition) {

    var value;

    if (condition) {
        value = "blue";

        // other codereturn value;
    } else {

        returnnull;
    }
}
```
value声明被移动（提升）到顶部而初始化仍旧保留在原来的地方，这意味着变量value在else从句中确实还是可获取到的，它只是有一个undefined的值因为还未被初始化。

对新的JavaScript开发者而言，他们总是需要花费一些时间来习惯声明提升，这种独特的行为最终会引发bug。因此，ECMAScript6引入了块级作用域的选项来确保更加强大的变量生命周期。

##### **Let 声明**

let声明语法与var相同。你基本上可以用let代替var来声明一个变量但将它的作用域保持在当前代码块，比如：
```javascript
function getValue(condition) {

    if (condition) {
        let value = "blue";

        // other codereturn value;
    } else {

        // value doesn't exist herereturnnull;
    }

    // value doesn't exist here
}
```
现在这个函数表现得更接近其他基于C的语言。用let代替var声明变量value。这意味着声明不会被提升到顶部，一旦执行离开if块变量value就会被销毁。如果condition的结果为false，则value从未被声明或初始化。

也许开发者最想要块级作用域变量的其中一个地方是for循环。不难看到这样的代码：
```javascript
for (var i=0; i &lt; items.length; i++) {
    process(items[i]);
}

// i is still accessible here and is equal to items.length
```
在其他语言中，块级作用域是默认的，这样的代码按照我们预期那样工作。在JavaScript中，变量i在循环结束之后仍旧可以访问因为var声明被提升了。使用let使你得到预期的行为：
```javascript
for (let i=0; i &lt; items.length; i++) {
    process(items[i]);
}

// i is not accessible here
```
在这段代码中，变量i只存在于for循环中，一旦循环完成，变量即被销毁，其他地方都不可访问。

 

##### **在循环中使用let**

let在循环中的行为同其他块略有不同。事实上每一个迭代都有使用自己的变量，而不是创建一个变量然后被循环的每一个迭代使用。这是为了解决JavaScript循环中的一个旧问题。考虑如下代码：
```javascript
var funcs = [];

 for (var i=0; i &lt; 10; i++) {
     funcs.push(function() { console.log(i); });
 }

 funcs.forEach(function(func) {
     func();     // outputs the number "10" ten times
 });
```
这段代码会连续10输出数字10，这是因为变量i在每次循环迭代中都是被共享的，也就是说循环中创建的闭包都引用相同的变量。一旦循环完成，变量i的值为10，所以这是每一个函数输出的值。为了修复这个问题，开发者在循环中使用立即调用函数表达式（IIFEs）强制创建变量副本：

 
```javascript
var funcs = [];

 for (var i=0; i &lt; 10; i++) {
     funcs.push((function(value) {
         returnfunction() {
             console.log(value);
         }
     }(i)));
 }

 funcs.forEach(function(func) {
     func();     // outputs 0, then 1, then 2, up to 9
 });
```
这个版本的例子在循环中使用了IIFE，变量i传递给IIFE，IIFE创建i的副本然后将它作为值存储起来，这是那个迭代中用于函数的值，因此调用每个函数都返回了预期的值。

let声明不用IIFE，它为你做了这些事情，每次迭代通过循环都会创建一个新的变量，并初始化一个同名的来自前一个迭代的变量的值。这意味着你可以通过下列代码来简化过程：
```javascript
var funcs = [];

 for (let i=0; i &lt; 10; i++) {}
     funcs.push(function() { console.log(i); });
 }

 funcs.forEach(function(func) {
     func();     // outputs 0, then 1, then 2, up to 9
 })
```
这段代码确实运行得与使用了var和IIFE的代码一样，但是，可以说，它更简洁。

不像var，let没有提升特性。一个用let声明的变量不可访问直到在let语句之后，试图这样做将导致引用错误：
```javascript
if (condition) {
    console.log(value);     // ReferenceError!
    let value = "blue";
}
```
这段代码中，变量vlaue是用let定义和初始化的，但是这个语句永远不会被执行因为前一行抛出了一个错误。任何时候你试图在定义之前使用相同块中的let变量也是这样。甚至正常地安全使用的typeof 也是不安全的。
```javascript
if (condition) {
    console.log(typeof value);     // ReferenceError!
    let value = "blue";
}
```
这里，正如前面的例子一样typeof value抛出了同样的错误。在同一个块中声明之前，你不能使用let变量。然而，你可以在块外使用typeof：
```javascript
console.log(typeof value);     // "undefined"if (condition) {
    let value = "blue";
}
```
这个例子在块外使用了typeof 操作符，块内声明了value。这意味着value未被绑定，typeof简单地返回了“undefined”。

如果标识符已经在这个块中被定义，在let声明中使用此标识符会导致抛出错误。比如：
```javascript
var count = 30;

// Syntax error
let count = 40;
```
这个例子中，count被声明两次，一次用var另一次用let。因为let不会重定义已经在同一域中存在的标识符，声明抛出了一个错误。如果let声明在域中创建了一个新的与包含块中变量同名的变量，就不会抛出错误。比如：
```javascript
var count = 30;

// Does not throw an errorif (condition) {

    let count = 40;

    // more code
}
```
这里let声明不会抛出异常因为它在if语句内创建了一个新的叫count的变量，新的变量覆盖了全局count，它（全局count）在if块中被阻止访问。

 

##### **全局let声明**

当在全局中使用let时有潜在的命名冲突因为全局对象有预定义的属性。使用let定义一个与全局对象属性同名的变量可能产生错误因为全局对象属性可能是不可配置的。因为let不允许在同一域中再定义同样的标识符，就不可能覆盖不可配置的全局属性。试图这样做会导致错误。比如：
```javascript
 let RegExp = "Hello!";          // ok
 let undefined = "Hello!";       // throws error
```
这个例子中第一行将全局的RegExp重定义为一个字符串，尽管这可能会有问题，不会抛出错误。第二行抛出了一个错误，因为undefined是不可配置的全局对象的自身属性。由于它的定义被环境锁定，let声明是不合法的。

在全局作用域中使用let并不常见，但如果你这样做了，了解这种情况是很重要的。

let的目标是长期取代var,因为前者的行为更接近其他语言的变量声明。如果你写的JavaScript只会在ECMAScript6或者更高的环境中运行，可能你想尝试只使用let，将var留给其他需要向后兼容的脚本。

注意：因为let声明在封闭块中不会将声明提升到顶部，你可能需要总是首先将let声明放在块中，以便它们在整个块中可用。

##### **常量声明**

另一个定义变量的新方式是使用const声明语法，使用const声明的变量被认为是常量，因此值一旦设置就不能改变。因此，每一个const变量必须被初始化。比如：
```javascript
// Valid constant
const MAX_ITEMS = 30;

// Syntax error: missing initialization
const NAME;
```
同let相似，常量也是块级声明。这意味着执行流一旦离开常量被声明且声明被提升到顶部的块，它们就会被销毁。比如：
```javascript
if (condition) {
    const MAX_ITEMS = 5;

    // more code}

// MAX_ITEMS isn't accessible here
```
这段代码中，常量MAX_ITEMS被声明在一个if语句中，一旦语句结束执行，MAX_ITEMS被销毁且在块外不可访问。

也同let相似的是，当一个const声明是由同一作用域中的已定义的变量标识符组成时，将会抛出一个错误。如果变量是由var（全局作用域或者函数作用域）或者let（块级作用域）声明的就没关系。比如：
```javascript
var message = "Hello!";
let age = 25;

// Each of these would cause an error given the previous declarations
const message = "Goodbye!";
const age = 30;

let与const最大的不同点在于，试图分配一个定义过的常量在严格模式和非严格模式下都会抛出错误。

const MAX_ITEMS = 5;

MAX_ITEMS = 6;      // throws error
```
警告：一些浏览器实现了const的pre-ECMAScript版本。实现的范围从简单的同义词var（允许值被重写）到实际上定义常量，但只是在全局或者函数作用域内。因此，在生产系统中使用const要特别小心。它可能无法为你提供你所预期的功能。

#### 解构赋值

JavaScript开发者花费了大量时间从对象和数组中取值。不难见到这样的代码：
```javascript
var options = {
        repeat: true,
        save: false
    };

// latervar localRepeat = options.repeat,
    localSave = options.save;
```
经常性地，为使代码更加简洁和更容易访问，对象属性被存储在本地的变量。ECMAScript6通过引入结构赋值使之更加容易，解构赋值系统地通过一个对象或数组，将指定的数据片段存储到本地变量中。

警告：如果结构赋值右边的值被解析为null或者undefined，将会抛出一个错误。

##### **对象结构**

对象解构赋值语法在赋值操作符发左边使用对象字面量。比如：

 
```javascript
var options = {
        repeat: true,
        save: false
    };

// latervar { repeat: localRepeat, save: localSave } = options;

console.log(localRepeat);       // true
console.log(localSave);         // false
```
 

这段代码中，options.repeat的值被存储在一个叫做localRepeat的变量中而options.save的值被存储在一个叫做localSave的变量中。这些都是指定使用对象字面量语法，键是在options中找到的属性，值是存储属性值的变量。

注意：如果给定名字的属性值不存在于对象中，本地的变量会得到undefined的值。

如果你想要使用属性名作为本地变量名，你可以省略冒号和标识符，比如：

 

 
```javascript
var options = {
        repeat: true,
        save: false
    };

// latervar { repeat, save } = options;

console.log(repeat);        // true
console.log(save);          // false
```
 

这里，两个叫做repeat和save的本地变量被创建，它们分别被初始化为options.value和options.save的值。当不需要有不同的变量名时，这段速记很有帮助。

结构也可以处理嵌套的对象，如下：
```javascript
var options = {
        repeat: true,
        save: false,
        rules: {
            custom: 10,
        }
    };

// latervar { repeat, save, rules: { custom }} = options;

console.log(repeat);        // true
console.log(save);          // false
console.log(custom);        // 10
```
在这个例子中，custom属性被嵌套在另一个对象中，花括号的额外设置允许你下行到嵌套的对象中然后获取其属性。

 

语法陷阱

如果你尝试使用不带var,let,const的结构赋值，你将会对结果感到吃惊：
```javascript
// syntax error
{ repeat, save, rules: { custom }} = options;
```
这会引起语法错误，因为左花括号通常是块的开始，而块不能作为赋值表达式的一部分。

解决办法是将左边的文字包裹在小括号里。
```javascript
// no syntax error
({ repeat, save, rules: { custom }}) = options;
```
这样运作就不会有任何问题。

##### **数组解构**

相似的，你可以在赋值操作符的左边使用数组字面量语法来解构数组。比如：
```javascript
var colors = [ "red", "green", "blue" ];

// latervar [ firstColor, secondColor ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```
这个例子中，数组解构从colors数组中取出第一个和第二个值，记住数组本身无论如何并不发生改变。

与对象解构类似，你也可以嵌套数组解构。只要用另一组方括号下行到子数组。
```javascript
var colors = [ "red", [ "green", "lightgreen" ], "blue" ];

// latervar [ firstColor, [ secondColor ] ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```
这里，secondColor引用了colors数组中“green”的值，这一项被包含在第二个数组中，所以在解构赋值中用额外的方括号包裹secondColor是很有必要的。

##### **混合解构**

在解构赋值中使用对象和数组的字面量将对象和数组混在一起是很有可能的。比如：
```javascript
var options = {
        repeat: true,
        save: false,
        colors: [ "red", "green", "blue" ]
    };

var { repeat, save, colors: [ firstColor, secondColor ]} = options;

console.log(repeat);            // true
console.log(save);              // false
console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```
这个例子中提取了两个属性值，repeat和save，还有两项来自colors数组的firstColor和secondColor。当然你也可以选择检索整个数组：
```javascript
var options = {
        repeat: true,
        save: false,
        colors: [ "red", "green", "blue" ]
    };

var { repeat, save, colors } = options;

console.log(repeat);                        // true
console.log(save);                          // false
console.log(colors);                        // "red,green,blue"
console.log(colors === options.colors);     // true
```
这个修改过的例子提取了options.colors,并将之存储在colors变量中，注意colors是一个对options.colors的直接引用而不是一个副本。

混合解构对不浏览整个解构而从JSON配置解构中取值非常有用。

#### 数值

因为整数和浮点数的单一类型双重用途，JavaScript的数值可以说特别复杂。数值用IEEE 754 双精度浮点格式存储，同样的格式用来存储两种类型的数值。作为JavaScript中的基本数据类型（和字符串、布尔一起）之一，数值对JavaScript开发者而言非常重要。JavaScript中游戏和图形成为新重点，ECMAScript6试图将数值处理得更加容易且更加强大。

##### ** 八进制与二进制字符串**

ECMAScript5想要通过在两个地方（parseInt和严格模式）移除以前包括的八进制整数表示法，来简化一些常见的数值误差。在ECMAScript3及更早版本，八进制数值用一个前置0后面跟任意数量的数字来表示。比如：
```javascript
// ECMAScript 3var number = 071;       
// 57 in decimalvar value1 = parseInt("71");    
// 71var value2 = parseInt("071");   
// 57
```
许多开发者会对这一版本的八进制字面量数字感到困惑，并因为对各个地方前置0误解的影响而产生许多错误。最令人震惊的是parseInt，如果有一个前置0，意味着这个值会被认为是八进制而不是十进制。这引出了Douglas Crockford的最基本规则之一：总是使用parseInt()的第二个参数来制定字符串应如何被解释。

ECMAScript5减少了八进制数字的使用。首先，parseInt()被改为如果没有第二个参数就忽略第一个参数中的前置0。这意味着数值再不会一不小心的被视为八进制了。第二个改变是在严格模式中消除了八进制表示法，试图在严格模式中使用八进制表示法将导致语法错误。
```javascript
// ECMAScript 5var number = 071;       
// 57 in decimalvar value1 = parseInt("71");        
// 71var value2 = parseInt("071");       
// 71var value3 = parseInt("071", 8);    
// 57function getValue() {
    "use strict";
    return 071;     // syntax error
}
```
通过做出这两个改变，ECMAScript5试图消除与八进制相关的大量混淆和错误。

ECMAScript6通过重新引入八进制表示法以及一个二进制表示法让事情更进一步。这些符号都用类似十六进制表示的值前面加上0x或者0X来暗示。新的八进制文本格式以0o或者0O开始，而新的二进制文本格式以以0b或者0B开始。每个字面量类型后面必须跟一个或者更多的数字，八进制是0-7之间，二进制是0-1之间。这里是一个例子：

// ECMAScript 6var value1 = 0o71;      // 57 in decimalvar value2 = 0b101;     // 5 in decimal

添加这两个文字类型使JavaScript开发者能够迅速且容易得引入二进制，八进制，十进制，十六进制格式的数值，这在数学运算的某些类型中是非常重要的。

parseInt()方法不再处理二进制或者八进制的字符串字面量：
```javascript
console.log(parseInt("0o71"));      // 0
console.log(parseInt("0b101"));     // 0
```
然而，Number函数会正确转换包含二进制或八进制字面量的字符串：
```javascript
console.log(Number("0o71"));      // 57
console.log(Number("0b101"));     // 5
```
当在字符串中使用八进制或二进制字面量时，确保了解你的使用情景,然后使用最合适的方法将他们转为数值。

##### **isFinate()与isNaN()**

JavaScript很久之前就有一对全局方法来识别某些特性的数值：

- 

isFinite() 决定一个值是否是有限值 (不是的话 

Infinity 或者 

-Infinity)
- 

isNaN() 决定一个值是否是NaN (因为 

NaN是唯一一个不等于自身的值)

尽管是为了处理数字，这些方法也可以从传入的任何值中推断出数值。这意味着如果传递的值不是数字时这两个方法都会返回错误的结果。比如：
```javascript
console.log(isFinite(25));      // true
console.log(isFinite("25"));    // true
console.log(isNaN(NaN));        // true
console.log(isNaN("NaN"));      // true
```
isFinate()和isNaN()都通过将参数传给Number()来得到一个数值，然后用它们来执行比较而不是用原始的值。如果在未检测值类型之前使用这其中一种方法的话，这一令人困惑的结果将导致错误。

ECMAScript添加了两个新方法来执行同样的比较，但是只用于数字类型：Number.isFinate()和Number.isNaN()。如果传入一个非数字的值，这两个方法总是返回false，如果传入数字时则返回同全局方法一样的值：
```javascript
console.log(isFinite(25));              // true
console.log(isFinite("25"));            // true
console.log(Number.isFinite(25));       // true
console.log(Number.isFinite("25"));     // false
console.log(isNaN(NaN));                // true
console.log(isNaN("NaN"));              // true
console.log(Number.isNaN(NaN));         // true
console.log(Number.isNaN("NaN"));       // false
```
这段代码中，Number.isFinate("25")返回false而isFinate("25")返回true。同样的，Number.isNaN("NaN")返回false而isNaN("NaN")返回true。

这两个方法旨在不显著改变语法来消除使用isFinate()和isNaN()处理非数值时可能引发的某些类型的错误。

**parseInt()和parseFloat()**

全局函数parseInt()和parseFloat()现在也被放在Number.parseInt()和Number.parseFloat()。这些函数表现得与全局同名函数完全一样，做出这一举动的唯一目的是对明显与具体数据类型相关的纯粹的全局函数进行分类。因为这些函数都是根据字符串创建数字，它们现在同其他与数值相关的方法一起归到Number。

##### **使用整数**

过去的这些年里，JavaScript的一个数值类型但是同时用来表示整数和浮点数已经引起了大量的混淆。尽管JavaScript已经尽最大的努力来确保开发者不需要担心细节，但是问题仍然时不时得泄露。ECMAScript6试图通过使之更容易识别和处理整数来解决这个问题。

**　　识别整数**

　　第一个是Number.isInteger()，你可以确定一个在JavaScript中显示的值是不是整数。因为整数和浮点数的存储不同。JavaScript引擎通过查看值的内部表现来确定。这意味　　着看起来像浮点数的数字也可以像整数一样被正确存储，因此Number.isInteger()返回true。比如：
```javascript
console.log(Number.isInteger(25));      // true
console.log(Number.isInteger(25.0));    // true
console.log(Number.isInteger(25.1));    // false
```
 

　　这段代码中，Number.isInteger对25和25.0都返回true，尽管后者看起来像一个浮点数。在JavaScript中，简单的给数字添加小数点并不会自动将之转为浮点数。因为25.0　　真的只是25，它是作为整数存储。而数字25.1作为浮点数存储，因为它有一个分数值。

　　**安全整数**

　　然而，整数并不都那么简单。JavaScript只能准确地显示 -253 到253之间的整数，超出这个安全范围，二进制表示法最终再利用多个数值。比如：
```javascript
console.log(Math.pow(2, 53));      // 9007199254740992
console.log(Math.pow(2, 53) + 1);  // 9007199254740992
```
　　这个例子不含错误。两个不同的数字最后显示了相同的JavaScript整数，值超出安全范围越远，影响越普遍。

　　ECMAScript6引入了Number.isSafeInteger()以更好地识别那些可以准确显示在JavaScript中的整数。Number.MAX_SAFE_INTEGER和Number.MIN_SAFE_INTEGER　　分别用以代表相同范围的上限和下限。Number.isSafeInteger()方法确保值是整数且值落在整数安全范围内：
```javascript
var inside = Number.MAX_SAFE_INTEGER,
    outside = inside + 1;

console.log(Number.isInteger(inside));          // true
console.log(Number.isSafeInteger(inside));      // true
console.log(Number.isInteger(outside));         // true
console.log(Number.isSafeInteger(outside));     // false
```
　　数字inside是最大的安全整数，所以它对Number.isInteger()和Number.isSafeInteger()都返回真，数字outside是第一个可疑的整数值，所以尽管它还是一个整数但是不再　　被认为是安全的。

　　大多数时候，你在JavaScript中做算术或比较的时候，仅仅只是想要处理安全整数。所以将它用作输入验证的一部分实在是一个好主意。

##### **新的Math方法**

上述JavaScript中游戏和图形的新重点使我们认识到许多数学计算用JavaScript引擎可以比纯JavaScript代码可以做得更高效。优化策略比如asm.js，致力于JavaScript的一个子集来提高性能，它需要更多的信息以尽可能快的方式进行计算。这很重要，比如，了解数字应该被视为32位整数还是64位浮点数。

因此，ECMAScript6为Math对象添加了几个新方法。这些新方法对提升常见数学计算的速度、提升那些必须提高计算的应用（比如图形程序）的速度很重要。下面是列出的新方法：

方法描述

 

Math.acosh(x) 返回x的反双曲余弦

 

Math.asinh(x) 返回x的反双曲正弦

 

Math.atanh(x) 返回x的反双曲正切

 

Math.cbrt(x) 返回x的立方根

 

Math.clz32(x) 返回32位整数表示中前导零的位数

 

Math.cosh(x) 返回x的双曲余弦

 

Math.expm1(x) 返回ex-1

 

Math.fround(x) 返回x的最近单精度浮点数

 

Math.hypot(...values) 返回的每个参数的平方和的平方根

 

Math.imul(x, y) 返回两个参数以32位整数形式相乘的结果

 

Math.log1p(x) 返回1 + x的自然对数  

 

Math.log10(x) 返回x的以10为底的对数

 

Math.log2(x) 返回x的以2为底的对数

 

Math.sign(x) 如果x是-0返回-1，如果是+0返回1

 

Math.sinh(x) 返回x的双曲正弦  

 

Math.tanh(x) 返回x的双曲正切

 

Math.trunc(x) 移除浮点数的小数部分，返回整数

详细解释每一个新方法及它们做了什么已超出本书范围。然而，如果你正在查找一个相当常见的计算，确保在你自己实现它之前检查新的方法。

 

#### 总结

ECMAScript6对JavaScript做了大量的大大小小的改变。一些在这章详述的小改变很可能被很多人忽视，但它们对语言演变的重要性同大改变是一样的。

完整的Unicode支持使得JavaScript能够以合乎逻辑的方式开始处理utf-16字符。通过codePointAt()和
String.fromCodePoint()在码点和字符之间转换的能力对字符串操作而言是重大的一步。增加的正则表达式u标识使得操作码点代替16位字符成为可能，而normalize()方法使得字符串比较更加恰当。

处理字符串的附加方法的增加，使你更易识别子字符串，不管它们有没有被找到，更多的功能被添加到正则表达式。Object.is() 方法对任何值执行严格相等，处理特殊的JavaScript值时，有效得成为===的安全版本。

let和const块绑定对JavaScript引入词法作用域。这些声明不会被提升且只存在它们被声明的那个块内。这意味着其行为更像其他语言，不太可能导致无意的错误，因为现在变量可以确切地声明在需要它们的地方。主要的JavaScript代码未来将专门使用let和const是很令人期待的，有效地使var成为语言中不推荐使用的部分。

通过引入新语法和新方法，ECMAScript使处理数字更加容易。二进制和八进制字面量格式使你可以直接将数字嵌入到源代码中，同时保持最适当的可见表现。Number.isFinite()和Number.isNaN()是同名全局方法的安全版本，由于它们缺乏强制类型转换。使用Number.isInteger()与Number.isSafeInteger()你可以更容易识别整数，Math上的新方法也使你可以做更多的数学运算。

尽管这些改变很小，它们将在未来几年内使JavaScript开发者的生活有显著差异。每一个改变都解决了一个特别令人关注的不改就需要大量自定义代码来解决的问题。通过在JavaScript中构建这些功能，开发者可以专注写产品代码而不是底层工具代码。

 