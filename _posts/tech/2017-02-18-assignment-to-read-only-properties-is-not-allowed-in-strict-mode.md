---
layout: post
title: Assignment to read-only properties is not allowed in strict mode
date:   2017-02-18 16:00:00 +0800
categories: angular 
tags: [angular, css, strict-mode]
---
## Angular模板要用``[attr.style]="..."``而不是``[style]="..."``

在AngularJS 2 template中使用``[style]="..."``时，Firefox/Chrome/Opera都认了，但MSEdge报错：

> Assignment to read-only properites is not allowed in strict mode

## style可能是只读对象

以下是出错的代码：

{% highlight ts linenos %}
//Angular组件
@Component({
  //踩坑
  template: `
    <div [innerHTML]="currentText" [style]="safeStyle"></div>
  `
})
export class ReadMoreComponent {

  @Input('style') unSafeStyle: string;

  private safeStyle: SafeStyle;

  constructor(private sanitizer: DomSanitizer) {
    this.safeStyle = 
      this.sanitizer.bypassSecurityTrustStyle(`this.unsafeStyle`);
  }
}
{% endhighlight %}

以下是正确的代码：
{% highlight ts linenos %}
//Angular组件
@Component({
  //跳坑
  template: `
    <div [innerHTML]="currentText" [attr.style]="safeStyle"></div>
  `
})
export class ReadMoreComponent {

  @Input('style') unSafeStyle: string;

  private safeStyle: SafeStyle;

  constructor(private sanitizer: DomSanitizer) {
    this.safeStyle = 
      this.sanitizer.bypassSecurityTrustStyle(`this.unsafeStyle`);
  }
}
{% endhighlight %}

区别在于：

{% highlight ts %}
[style]="..." //踩坑
[attr.style]="..." //跳坑
{% endhighlight %}

这只是Angular2的写法，深层的原因与HTMLElement.style对象有关。

## 设置style属性的方法

**别用``element.style="..."``修改样式！因为``element.style``可能是只读的**（如果代码在遵循规范的浏览器中运行，``HTMLElement.style``是一个只读``CSSStyleDeclaration``对象）。

IE严格遵循了这条规范，Firefox/Chrome/Opera却一起放水，允许使用``element.style="..."``。我们一般不用IE进行开发和调试，如果在部署前没有针对IE充分测试，又认不得这个坑，就踩进去了。

正确的做法有三种：

{% highlight javascript %}

//设置单个样式
element.style.color = "white";
element.style.backgroundColor = "black";

//设置全部样式
element.style.cssText = "color:white;background-color:black;"

//通过改写style属性来设置全部样式
element.setAttribute('style',"color:blue;background-color:black;")

{% endhighlight %}

element.style属性是个只读对象，但访问style对象的自身属性，能设置单个样式（如style.color），也能设置全部样式（style.cssText），或者彻底改写style（element.setAttribute）。

``element.style="..."``最符合直觉——**学习的目的之一是“纠正那些不正确的直觉”**。

[深入了解 ``HTMLElement.style`` >>](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/style)

## 用Node.js做个验证

为了验证“只读属性不能赋值”，可以用Node.js分别在严格模式和普通模式做个验证：

{% highlight javascript %}
//正常模式
//node
var obj = {};
Object.defineProperty(obj, 'prop', 
  {
    value: 'a', 
    writable: false
  }
);
obj.prop = 'b'

console.assert(obj.prop == 'a'); //虽然不报错，但prop并未被成功设置，仍然等于'a'
{% endhighlight %}

{% highlight javascript %}
//严格模式
//node --use_strict
var obj = {};
Object.defineProperty(obj, 'prop', 
  {
    value: 'a', 
    writable: false
  }
);
obj.prop = 'b'
// TypeError: Cannot assign to read only property 'prop' of object ...
{% endhighlight %}

**注意**：之前那些放水（也可说“兼容性较好”）的浏览器是处于严格模式下的，而且，他们让“赋值给只读属性”这个操作在严格模式下正确执行，而这里我们看到，在正常模式下，“赋值给只读属性”这个操作既不会报错，也得不到预期结果。

## 可能通过设置让浏览器严格遵循ECMA规范吗？
如果Firefox/Chrome/Opera们在严格模式下采用更“严格”的策略来执行脚本（即遵循ECMA规范），那类似的问题就基本能够在程序调试期间被提前避免。
