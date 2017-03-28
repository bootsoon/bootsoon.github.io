---
layout: post
title: 间接实现Angular2+IE9上传文件
categories: angular2
tags: [IE9 FileUpload FileAPI FormData XMLHttpRequest-Level-2]
---

Angular2编写文件上传功能时会遇到一个困难：IE9仅支持用HTML表单上传文件。

我们没法在IE9里直接实现“异步上传一个文件”。

当然我们能找到方法：构造隐藏的iframe+`HTML4`表单，或者其它技巧。

但我遇到的场景却迫使我采用了最笨的方法：跳转到HTML4网页，完成上传后再跳转回ng2应用。

## IE9只能用表单提交文件

因为IE9不支持以下特性：

* IE9不支持FileAPI(IE10开始支持) [Can I use FileAPI](http://caniuse.com/#search=fileAPI)
* IE9不支持FormData(IE10开始支持) [Can I use FormData](http://caniuse.com/#search=FormData)
* IE9不支持XMLHttpReqeust2传输文件内容(IE10开始支持) [Can I use XMLHttpRequest](http://caniuse.com/#search=XMLHttpRequest)

这些都是异步传输文件需要的特性，垫片和补丁填补不了IE9缺失的这些特性。

## 遇到IE9时可采用的方法

* 方法1：flash
  
  * 优点：体积很小的flash就可以达到目的，编程简单，使用体验很好，服务端API可以通用。
  * 缺点：只有不足6%的网站还在使用flash，用户极有可能根本不需要flash插件，强制用户安装一个他们不使用的浏览器插件。

* 方法2：iframe 

  动态构造一个隐藏的iframe+HTML4 form，这也是`blueimp-file-upload`在遇到IE9时采用的方法。
  * 优点：可以借助成熟的`blueimp-file-upload`，编程简单，用户体验很好，服务端API可以通用。
  * 缺点：`blueimp-file-upload`依赖`jQuery`，但如果你已经用了`JQuery`，那没关系。


* 方法3：跳转到HTML4表单页面

  * 优点: 
    * 避开了iframe的安全问题
    * 没有了对flash插件的依赖，也不需要更多库的支持，只需要一个或多个传统页面
  * 缺点：
    * 需要在服务端编写一个甚至多个专门用来接受文件的HTML4表单页面
    * 用起来不方便，要从当前应用跳出去，完事儿之后又要跳回来
    * 跳转出去前的状态保存和跳转回来后的状态恢复很麻烦
    * 要分别为HTML5和HTML4各写一份代码
   
* 方法4：用Modal对话框+HTML4表单页面

  实现与方案3类似，用户不需要离开应用，应用也不需要费力去保存和恢复状态，只要细节处理得当，用户体验会好很多。

  * 优点: 
    * 避开了iframe的安全问题
    * 没有了对flash插件的依赖，也不需要更多库的支持，只需要一个或多个传统页面
    * 用起来还算方便，无需跳转
    * 省去了跳转相关的状态保存和恢复的麻烦
  * 缺点：
    * 需要在服务端编写一个甚至多个专门用来接受文件的HTML4表单页面
    * 要分别为HTML5和HTML4各写一份代码
   
## 我遇到的问题

  但我遇到了一个挺特殊的问题，我的网页被预设的场景是一定会运行在内嵌IE中，而这个内嵌的IE属于其它厂商制作的软件的一部分（已停止更新但仍在使用）。
  
  这些内嵌IE主要是IE9和IE10，支持这个内嵌IE浏览器是必须被满足的要求，同时，也必须其它支持HTML的独立浏览器。
  
  我原本选择的是方法2，编写很顺利，但在测试时遇到了解决不了的问题。
  
  在提交文件到服务端后接收返回值的过程中，如果服务端正常接收了文件并返回2XX，那么一切正常。
  
  但如果服务端判定所提交的文件不合法（比如文件尺寸超限、文件类型不合法），服务端会返回状态码为4XX的含有错误提示信息的JSON文件，供浏览器解析以提示用户。
  
  运行在内嵌IE9中的iframe收到JSON文件时，浏览器弹出“文件下载-安全警告”，大致内容和下面这样：

  {% highlight text %}
            文件下载-安全警告

  是要保存此文件，还是要联机查找程序来打开次文件？

        名称： http_4_webOC
        类型： 未知文件类型，6.10KB
        来源： ieframe.dll
                          查找  保存  取消
  {% endhighlight %}

  程序得不到解析JSON返回值的机会，用户看不到预设的错误提示，看到的是“安全警告”，且无论作何选择，我们的代码都没有机会再运行了。
  
  用户一定觉得“程序坏了”，而且“安全警告”显然让用户觉得“这网站不安全”。事实上程序也确实坏了，至少这个文件上传功能坏掉了。
 
  这让我不得已放弃了方法2，最后选择了方法3。
  
  方法4是我完成项目之后才想到的，所以，如果以后遇到，我会用方法4。


## 代码概要

### 1. 判断是不是运行在IE9里

有很多方法可以判断，我觉得下面这方法比较简单。

{% highlight typescript %}
import { Component, Renderer, Inject } from '@angular/core';
import { DOCUMENT } from '@angular/platform-browser';

@Component({
  //...
})
export class TopicEditComponent {

  constructor(
    private renderer: Renderer,
    @Inject(DOCUMENT) private document: any
  ) { }

  public get isIE9(): Boolean {
    // documentMode is an IE-only property
    // http://msdn.microsoft.com/en-us/library/ie/cc196988(v=vs.85).aspx
    return this.document.documentMode === 9;
  }
  //...
{% endhighlight %}


### 2. 根据浏览器决定弹框还是跳转

{% highlight html %}
<!-- 隐藏的File Input -->
<input #upload type="file" ng2FileSelect [uploader]="uploader" multiple style="display:none;" />
<!-- 点击后可能跳转，也可能触发#upload的click事件 -->
<a role="button" (click)="openPickerOrRedirect()">添加图片</a>
{% endhighlight %}

{% highlight typescript %}
  //...

  @ViewChild('upload') upload: ElementRef;

  public redirectToUploadingUrl = () => {
    window.location.href = this.service.getTopicPictureUploadRedirectUrl();
  }

  public openPickerOrRedirect = () => {
    if (this.isIE9) {
      this.redirectToUploadingUrl();
    } else {
      // trigger the click event of the hidden input#upload
      this.renderer.invokeElementMethod(this.upload.nativeElement, 'click');
    }
  }

  //...
{% endhighlight %}

## 参考链接

* [MDN - Using files from web applications](https://developer.mozilla.org/en-US/docs/Using_files_from_web_applications)
* [MDN - Using XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest)
* [Github - blueimp-file-upload (jQuery File Upload Plugin)](https://github.com/blueimp/jQuery-File-Upload)
* [SO - IE9 prompts user on submission of hidden iframe](http://stackoverflow.com/questions/9230779/IE9-prompts-user-on-submission-of-hidden-iframe)
