---
title: Strict Mode 总结
---

# Strict Mode 是什么？

[MDN: Strict mode](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode)

EC5严格模式是一种可选的限制性JS变种。严格模式不只是一个语言子集，它与标准代码有不同语义。不支持严格模式的浏览器与支持严格模式的浏览器以不同的方式运行严格模式代码，严格模式和非严格模式代码可以共存。

严格模式对标准JS的语义做了几个改变：首先，针对一些JS错误，不再沉默，改为引发，从而消除它们。其次，修正了一些导致JS引擎难以优化性能的错误，相同的代码在严格模式下经过引擎的合理优化，通常要比在非严格模式下执行得更快。最后，严格模式禁用了某些有可能在未来版本的ECMA脚本中定义的语法。

有时能看到符合规范的非严格模式被称为"懒散模式"(sloppy mode)，这不是官方术语，但应该有所了解。

## 启用严格模式

严格模式可以被应用到整个脚本或独立函数。在eval代码,function代码,事件句柄，传递给setTimeout的字符串或者在整个脚本中使用严格模式都是可行的。

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

严格模式把一些可接受的mistakes改成了errors，JS原先被设计为新手易于使用的语言，一些理应引发异常的操作，没有引发异常。有时会制造更大的麻烦。严格模式把这些mistakes视为errors，好让他们能够被发现并且及时修正。

#### 1. 严格模式不允许随意创建全局变量，在标准JS中，在赋值语句中错拼了变量名会创建一个新的全局属性，然后，代码会继续“工作”。那些会随意创建全局变量的赋值语句在严格模式下会抛异常。

{% highlight javascript linenos %}
'use strict';
                       // Assuming a global variable mistypedVariable exists
mistypedVariable = 17; // this line throws a ReferenceError due to the 
                       // mispelling of variable
{% endhighlight %}

#### 2. 严格模式让那些之前会失败并沉默异常的语句抛出异常

例如，NaN是一个不可写的全局变量，在标准JavaScript代码中，赋值给NaN的不会有啥影响，开发人员得不到失败反馈。在严格模式下，赋值给NaN会抛出异常。所有在标准代码中赋值失败然后沉默的语句都会在严格模式下抛出异常。

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

在标准模式下，后一个名称重复的参数遮蔽了前一个相同名称的参数，前一个参数还可以通过arguments[i]来访问，因为他们并不是完全不能访问。这种遮蔽有它的用处，但可能不是我们想要的，因为它可能隐藏着拼写错误，所以在严格模式下，重复的参数名称是语法错误。

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

with语句的问题在于，任何处于with代码块中的变量名既可能是对象的属性，也可能是外层作用域的变量（甚至是全局变量）。在程序运行状态时，没办法提前知道。严格模式把with语句变成一个语法错误，因此，在程序运行时，一个变量没有机会在with语句块中指向一个未知位置。

{% highlight javascript linenos %}
'use strict';
var x = 17;
with(obj){ // !!! syntax error
  // If this weren't strict mode, would this be var x, or
  // would it instead be obj.x? It's impossible in general
  // to say without running the code, so the name can't be
  // optimized.
  x;
}
{% endhighlight %}

把对象赋值给一个短名称变量，然后访问这个变量的相应属性的简单替代方法，还没有准备好。

#### 2. 严格模式下的eval语句不会引入向当前作用域引入新变量

标准模式下，eval("var x;") 引入一个x变量到当前函数或全局作用域。这就是说，一般情况下，在程序运行时，在一个包含eval调用的函数中，每个不指向函数参数的局部变量必须被映射到一个特殊的定义（因为eval可能引入了一个新变量，这个新变量会隐藏外层变量）。在严格模式下，eval只为被求值的语句创建变量，因此不影响一个变量名称指向的是一个外层变量还是某个局部变量。

{% highlight javascript linenos %}
var x = 17;
var evalX = eval("'use strict'; var x= 42; x;");
console.assert(x === 17);
console.assert(evalX = 42);
{% endhighlight %}

相应的，如果在严格模式下通过表达式eval(...)调用eval函数，代码会按照严格模式来求值。尽管代码可以显式地声明严格模式，但也不必要这样做。

{% highlight javascript linenos %}
function strict1(str){
  'use strict';
  return eval(str); // str will be treated as strict mode code
}
function strict2(f, str){
  'use strict';
  return f(str); // not eval(...): str is strict if and only
                 // if it invokes strict mode
}
function nonStrict(str){
  return eval(str); // str is strict if and only
                    // if it invokes strict mode
}
strict1("'Strict mode code!'");
strict1("'use strict'; 'Strict mode code!'")
strict2(eval, "'Non-strict code.'");
strict2(eval, "'use strict'; 'Strict mode code!'");
nonStrict("'Non-strict code.'");
nonStrict("'use strict'; 'Strict mode code!'");
{% endhighlight %}

Thus 
names
  in strict mode eval code 
behave 
identically 
  to names 
       in strict mode code not being evaluated as the result of eval.

？ 这样的话，严格模式下，eval代码中的name与那些不作为eval调用结果的name表现出相同的行为。

eval本身的古怪行为，也值得探究一番：

{% highlight javascript linenos %}
var x = 'outer';
(function() {
  var x = 'inner';
  eval('console.log("direct call: " + x)'); 
  (1,eval)('console.log("indirect call: " + x)'); 
})();
{% endhighlight %}

{% highlight bash %}
direct call: inner
indirect call: outer
{% endhighlight %}

不理解啊！啥意思？


#### 3. 严格模式禁止删除plain names， delete name在严格模式是个语法错误

{% highlight javascript linenos %}
'use strict';
var x;
delete x; // !!! syntax error

eval('var y; delete y;'); // !!! syntax error
{% endhighlight %}

### eval和arguments变得更简单

严格模式使得arguments和eval不那么奇怪。在标准代码中，这两者都牵扯到大量奇怪的行为：eval会增加或删除绑定，还会改变绑定的值，arguments用它的索引属性来给形参添加了别名。严格模式把eval和arguments视为关键字，这是长足的进步，尽管完整的补丁在未来版本的ESMAScript中才会有。

#### 1. eval和arguments不能被绑定或赋值，这样做是语法错误

{% highlight javascript linenos %}
'use strict';
eval = 17;
arguments++;
++eval;
var obj = { set p(arguments) {} };
var eval;
try {} catch (arguments) {}
function x(eval) {}
function arguments() {}
var y = function eval() {};
var f = new Function('arguments', "'use strict'; return 17;");
{% endhighlight %}

#### 2. 严格模式不会把函数内部创建的arguments对象的属性别名化。

标准模式下，在函数内，如果arg是第一个参数，设置arg也就是设置arguments[0], 反之亦然，除非没有参数被传递或者arguments[0]被删掉了。严格模式下，当函数被调用时，函数保存arguments原始值，arguments[i]不跟随对应的形参，形参也不跟随对应的arguments[i]。

{% highlight javascript linenos %}
function f(a){
  'use strict';
  a = 42;
  return [a, arguments[0]];
}
var pair = f(17);
console.assert(pair[0] === 42);
console.assert(pair[1] === 17);
{% endhighlight %}

#### 3. 不再支持 arguments.callee 

在标准代码中，arguments.callee指向封装函数，这个用法很脆弱，简单地命名封装函数。此外，arguments.callee在很大程度上妨碍了优化，比如内联函数，因为，如果arguments.callee被访问乐，就必须被提供一个指向非内联函数的引用。严格模式函数的arguments.callee是一个non-deletable属性，存取它会抛出异常。

{% highlight javascript linenos %}
'use strict';
var f = function() {
  return arguments.callee;
};
f();
{% endhighlight %}

### 更安全的JavaScript

严格模式使得我们更易于写出安全的JavaScript。有些网站目前允许用户写JavaScript代码，网站替其它用户运行这些代码。JavaScript在浏览器中可以访问用户隐私，因此这样的JavaScript代码在运行前，必须被部分改造，以审查是否存在对禁用功能的访问。如果没有很多运行时检查，JavaScript的灵活性使得这样做实际上是不可能的。这样的语言函数太多以至于执行运行时检查有着显著的性能损失。几个严格模式的小技巧，

A few strict mode tweaks, plus requiring that user-submitted JavaScript be strict mode code and that it be invoked in a certain manner, substantially reduce the need for those runtime checks.

#### 1. 不再强制传递给this的值是一个对象。

对于标准模式的函数，this总是一个对象，无论用一个对象this；如果用一个布尔this，字符串this或者数字this，

如果用对象来调用，this就是提供的对象；
如果用Boolean,string或number的this来调用，this就是提供的变量值；
如果用undefined或null的this来调用，this就是global对象。

可以用call, apply, 或者bind来指定一个特定的this。不仅自动装箱是一个性能损失，而且在浏览器中暴露全局对象也是一个安全隐患，因为全局对象提供了那些功能的访问，这些功能是“安全的”JavaScript环境必须限制的。因此，对于一个严格模式函数，指定的this不被装箱成一个对象，如果没有指定this，this就是undefined:

{% highlight javascript linenos %}
'use strict';
function fun() { return this; }
console.assert(fun() === undefined);
console.assert(fun.call(2) === 2);
console.assert(fun.apply(null) === null);
console.assert(fun.call(undefined) === undefined);
console.assert(fun.bind(true)() === true);
{% endhighlight %}

这意味着，严格模式下，不再能通过this指向window对象。

#### 2. 严格模式下，不能通过标准实现的ECMAScript扩展来遍历JavaScript堆栈。

在标准模式下，当函数fun正在被调用时，fun.callee是那个最近调用fun的函数，fun.arguments是那次调用的arguments。这两个扩展对于安全的JavaScript来说都是有问题的，因为它们允许“安全的代码”访问“特权”函数和这些函数的潜在的部安全的参数。如果fun放在严格模式下，fun.caller和fun.arguments都是non-deletable属性，存取它们会报错。

{% highlight javascript linenos %}
function restricted() {
  'use strict';
  restricted.caller; // throws a TypeError
  restricted.arguments; // throw a TypeError
}
function privilegedInvoker(){
  return restricted();
}
privilegedInvoker();
{% endhighlight %}


#### 3. 严格模式下的arguments不再提供对相应的函数调用的变量的访问。在同样的ECMAScript实现中，arguments.caller是一个对象，这个对象的属性别名化了这个函数中的变量。这是一个安全隐患，因为它破坏了通过函数抽象来隐藏特权变量的能力。它还妨碍了大部分优化。考虑到这些理由，没有最近的浏览器实现它。但是由于它的历史功能，严格模式下的arguments.caller也是一个non-deletable属性，读写它都会报错。

{% highlight javascript linenos %}
'use strict';
function fun(a, b){
  'use strict';
  var v = 12;
  return argument.caller; // throws a TypeError
}
fun(1, 2); // doesn't expose v (or a or b)
{% endhighlight %}


### 为ECMAScript的未来版本做好准备

未来的ECMAScript版本可能引入新语法，ECMAScript 5的严格模式下使用了一些限制，以便顺利过度。

#### 1. 严格模式下一小部分标识符变成了保留字。

这些保留字是：implements, interface, let, package, private, protected, public, static和yield。在严格模式下，你不能命名和使用这些保留字。

{% highlight javascript linenos %}
function package(protected){ // !!!
  'use strict';
  var implements; // !!!
  
  interface:  // !!!
  while (true) {
    break interface; // !!!
  }

  function private() { } // !!!
}
function fun(static) { 'use strict' } // !!!
{% endhighlight %}


#### 2. 严格模式禁止非顶级的函数声明。

浏览器的标准模式下，函数声明可以在任何地方。（这不符合ES5甚至ES3!）这是一个不同浏览器的兼容性扩展。未来的ECMAScript版本有希望未非顶层的函数声明定义新的函数声明语义。禁止此类函数声明为将来版本的ECMAScript的语言规范扫除障碍。


{% highlight javascript linenos %}
'use strict';
if (true) {
  function f() {} // !!! syntax error
  f();
}

for (var i = 0; i < 5; i++) {
  function f2() {} // !!! syntax error
  f2()
}

function baz() { // kosher
  function eit() {} //kosher
}
{% endhighlight %}

这种禁止并不适合严格模式，因为这种函数声明是基础ES5的扩展。但这是ECMAScript委员会推荐的做法，浏览器会实现它。

### 浏览器的严格模式

现在主流浏览器都实现了严格模式。但也不要盲目地依赖它，因为仍然有很多还在使用的浏览器版本仅支持部分严格模式，或者支持
得不完整（例如，IE10之前的版本）。严格模式改变语义。在未实现严格模式的浏览器中，依赖这些改变会引起错误。谨慎地使用严格模式，如果你只在不支持严格模式的浏览器下测试，很有可能在支持严格模式下的浏览器上遇到问题，反之亦然。


# 本文引出的问题：

* mistake vs. error

  mistake:  If you make a mistake, you do something which you did not intend to do, or which produces a result that you do not want.

  erro: An error is something you have done that is considered to be incorrect or wrong, or that should not have been done. 

* syntax vs. semantic

  syntax:  Syntax is the set of rules that describes how a computer language can be used to make programs.

  semantic: Semantic is used to describe things that deal with the meanings of words and sentences. 
