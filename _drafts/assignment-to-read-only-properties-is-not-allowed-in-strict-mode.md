---
title: "在AngularJS 2 template中使用...[style]=\"safeStyle\"...时，MSEdge报错...\"Assignment to read-only properites is not allowed in strict mode\""
---
# 起

当红色的错误在控制台里跳动，不免又怀疑是Edge/IE系列的错，一番探究之后，剧情反转，再次警醒自己——“一切都是你的错”。

# Strict Mode 是个什么鬼？

[Strict mode](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode)

ECMAScript 5 的Strict mode(严格模式)是一种选择加入一种JavaScript受限变种的方法。严格模式并不不仅仅是一个语言子集。它有意区别于普通代码而有着不同语义。不支持Strict mode的浏览器与支持Strict mode的浏览器以不同的方式运行Strict mode代码。strict mode和non-strict mode代码可以共存。

Strict mode makes several changes to normal JavaScript semantics. 

strict mode针对普通JavaScript的语义做了几个改变：

* 首先，把一些JavaScript异常从沉默改为抛出，消灭这些错误。
* 其次，修正了一些导致让JavaScript引擎难以优化性能的错误，这使得strict mode代码通常比同等的non-strict mode执行得更快。
* 最后，strict mode禁用了一些语法可能在未来版本的ECMAScript中定义的语法。

有时候，能看到标准的、non-strict模式被称为"sloppy mode"(马大哈模式)，这不是官方术语，但也应该有所了解，以避免被蒙圈。

## 启用严格模式

可以把严格模式应用到整个脚本或独立函数中。它不会应用到用{}包裹的语句块中，eval,function,event句柄属性，传递到setTimeout()的字符串，和


### 脚本级

{% highlight JavaScript linenos %}
//Whole-script strict mode syntax
'use strict';
var v = "Hi! I'm a strict mode script!";
{% endhighlight %}

### 函数级

{% highlight JavaScript linenos %}
function strict(){
  //Function level strict mode syntax
  'use strict';
  function nested() {
    return 'And so am I!';
  }
  return "Hi! I'm a strict mode function! " + nested();
}
function nonStrict(){
  return "I'm not strict."
}
{% endhighlight %}

## 严格模式下的几个改变

严格模式改变语法和运行时行为。这些变化总的来说有这几个方面：

### 把mistakes变成errors

严格模式把一些可接受的mistakes改成了errors，JavaScript原先是被设计成易于新手使用的语言，有时候一些应该引发异常的操作，被赋予了不会引发异常语法。有时这能解决眼前的问题，但也有时会制造更大的麻烦。严格模式把这些mistakes视为errors，好让他们能够被发现并且及时修正。

#### 1. 严格模式不允许随意创建全局变量，在常规JavaScript中，在赋值语句中错拼了变量名会创建一个新的全局属性，然后，代码会继续“工作”。那些会随意创建全局变量的赋值语句在严格模式下会抛异常。

{% highlight javascript linenos %}
'use strict';
                       // Assuming a global variable mistypedVariable exists
mistypedVariable = 17; // this line throws a ReferenceError due to the 
                       // mispelling of variable
{% endhighlight %}

#### 2. 严格模式让那些之前会失败并沉默异常的语句抛出异常

例如，NaN是一个不可写的全局变量，在常规JavaScript代码中，赋值给NaN的不会有啥影响，开发人员得不到失败反馈。在严格模式下，赋值给NaN会抛出异常。所有在常规代码中赋值失败然后沉默的语句都会在严格模式下抛出异常。

* 赋值给不可写属性(non-writable property)
* 赋值给只读属性(getter-only property)
* 给不可扩展对象添加新属性(non-extensible object)


{% highlight javascript linenos %}
'use strict';
// Assignment to a non-writable properity
var obj1 = {};
Object.defineProperty(obj1, 'x', {value: 42, writable: false});
obj1.x = 9; // throw a TypeError

// Assignment to a getter-only property
var obj2 = { get x() { return 17; }};
obj2.x = 5; // throws a TypeError

// Assignment to a new property on a non-extensible object
var fixed = {};
Object.preventExtension(fixed);
fixed.newProp = 'ohai'; // throws a TypeError
{% endhighlight %}

#### 3. 严格模式会让那些要删除“不能删除的属性”的尝试抛出异常（以前只是没效果）

{% highlight javascript linenos %}
'use strict';
delete Object.prototype; // throws a TypeError
{% endhighlight %}

#### 4. 严格模式要求函数的参数名称必须唯一

在常规模式下，后一个名称重复的参数遮蔽了前一个相同名称的参数，前一个参数还可以通过arguments[i]来访问，因为他们并不是完全不能访问。这种遮蔽有它的用处，但可能不是我们想要的，因为它可能隐藏着拼写错误，所以在严格模式下，重复的参数名称是语法错误。

{% highlight javascript linenos %}
function sum(a, a, c) { // !!! syntax error
  'use strict';
  return a + b + c; // wrong if this code ran
}
{% endhighlight %}

#### 5. ECMAScript 5严格模式禁用十进制语法(octal syntax)。十进制语法不属于ECMAScript 5，但所有的浏览器都支持以"0"为前缀的十进制数值定义（例如 0644 === 420 和 "\045" === "%"）。ECMAScript 5中，十进制数字加前缀"0o"

{% highlight javascript linenos %}
var a = 0o10; // ES2015: Octal
{% endhighlight %}

新丁有时误以为前缀0没有语义，并用它来对齐——但这样做改变了数字的数值！十进制语法不仅没用还会被误用，所以严格模式让十进制语法报错：

{% highlight javascript linenos %}
'use strict';
var sum = 015 + // !!! syntax error
          197 +
          142;
{% endhighlight %}

#### 6. ECMAScript 2015的严格模式禁止为原生类型值(primitive values)设置属性，如果不使用严格模式，设置属性的行为只是被简单忽略，使用严格模式，会抛出TypeError。

{% highlight javascript linenos %}
(function(){
  'use strict';

  false.true = '';          // TypeError
  (14).sailing = 'home';    // TypeError
  'with'.you = 'far away';  // TypeError
})();

{% endhighlight %}

### 简化变量使用

严格模式简化了代码中变量名映射到变量定义的过程，很多编译器的优化依赖一种能力，它能告诉编译器变量X存放的位置，这对于优化JavaScript代码至关重要。JavaScript有时直到运行时才确定变量名称到变量定义的绑定关系。严格模式删除乐大部分这种情况，由此编译器可以更好地优化代码。

#### 1. 严格模式禁用with语句

with语句的问题在于，任何处于with代码块中的变量名既可能是对象的属性，也可能是外层scope的变量（甚至是全局变量）。
