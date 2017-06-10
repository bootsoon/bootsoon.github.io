JIT损失性能

* 因为需要在浏览器里编译模板这个步骤，视图需要更长时间来显示。
* 因为要加入编译器及相关代码，应用程序体积更大，需要更长时间来传输和加载。

编译能发现许多组件模板绑定错误...

AOT和JIT的差异仅在也编译的时机和工具，它们使用同一个Angular编译器。

AOT的好处

* 渲染得更快
* 减少了异步请求的脚本数量
* 减少了必需的Angular框架代码（不再“编译器”这部分代码）
* 提前检查模板错误
* 提高了安全性

AOT要求@Component的模板文件和样式文件的路径必须是相对的，JIT没有这样的要求。


Tree Shaking

Rollup



AoT Don'ts

https://github.com/AngularClass/angular2-webpack-starter#aot-donts

The following are some things that will make AoT compile fail.

* Don’t use require statements for your templates or styles, use styleUrls and templateUrls, the angular2-template-loader plugin will change it to require at build time.
* Don’t use default exports.
* Don’t use form.controls.controlName, use form.get(‘controlName’)
* Don’t use control.errors?.someError, use control.hasError(‘someError’)
* Don’t use functions in your providers, routes or declarations, export a function and then reference that function name
* @Inputs, @Outputs, View or Content Child(ren), Hostbindings, and any field you use from the template or annotate for Angular should be public



Making your Angular 2 library statically analyzable for AoT

https://medium.com/@isaacplmann/making-your-angular-2-library-statically-analyzable-for-aot-e1c6f3ebedd5#.24c67on3c

给出了几个错误原因、编译器的提示和相应的解决办法：

1. const lambda => export function
1. default export => named export
1. private, protected accessors should be changed to public for any members accessed from template
1. dynamic component template => static template
1. moduleId should be set on components with templateUrl

> const lambda => export function   
> Error encountered resolving symbol values statically. Calling function ‘declarations’, function calls are not supported. Consider replacing the function or lambda with a reference to an exported function

> default export => named export  
> can't resolve module ./some.component from /path/to/some.module.ts Error: can't find symbol undefined exported from module /path/to/some.component.ts

> private, protected accessors should be changed to public for any members accessed from template
> Error at /path/to/some.component.ngfactory.ts:134:40: Property 'context' is private and only accessible within class 'SomeComponent'.

> dynamic component template => static template  
> Error encountered resolving symbol values statically. Expression form not supported

> Unreproduced

> Error: (when AoT compiling an app that uses the library)  
> Uncaught Error: Unexpected value 'undefined' imported by the module 'AppModule'


angular2-image-popup

不支持aot编译，ng-bootstrap有个modal组件，仔细阅读文档和示例后，发现有个很好的方法，其open方法可以接受另一个angular组件，这样我就能比较简单又灵活地解决“点击缩略图后显示原图”的操作了。

## AOT vs. JIT

实践对比：

项目规模： 

Chrome DevTools Timeline panel

You may see one to three dotted, vertical lines on your Flame Chart. The blue line represents the DOMContentLoaded event. The green line represents time to first paint. The red line represents the load event.

### JIT

红线 Load event at 3.68s
蓝线 DOMContentLoaded event at 3.68s
绿线 first paint 681ms

### AOT

红线 Load event at 1.23s
蓝线 DOMContentLoaded event at 1.22s
绿线 first paint 274ms



before:

    "@angular/common": "2.2.3",
    "@angular/compiler": "2.2.3",
    "@angular/core": "2.2.3",
    "@angular/forms": "2.2.3",
    "@angular/http": "2.2.3",
    "@angular/platform-browser": "2.2.3",
    "@angular/platform-browser-dynamic": "2.2.3",
    "@angular/router": "3.2.3",
    "@ng-bootstrap/ng-bootstrap": "^1.0.0-alpha.18",
    "@ngrx/core": "^1.2.0",
    "@ngrx/store": "^2.2.1",
    "angular-confirmation-popover": "^2.0.3",
    "angular2-image-popup": "1.1.1",
    "angular2-moment": "1.0.0",
    "blueimp-file-upload": "9.14.2",
    "bootstrap": "4.0.0-alpha.5",
    "core-js": "^2.4.1",
    "font-awesome": "^4.6.3",
    "jquery-ui": "^1.12.1",
    "ng2-cookies": "^1.0.2",
    "ng2-page-scroll": "^4.0.0-beta.2",
    "ng2-slim-loading-bar": "^2.0.4",
    "ng2-toastr": "^1.3.2",

after:

    "@angular/common": "^2.4.8",
    "@angular/compiler": "^2.4.8",
    "@angular/core": "^2.4.8",
    "@angular/forms": "^2.4.8",
    "@angular/http": "^2.4.8",
    "@angular/platform-browser": "^2.4.8",
    "@angular/platform-browser-dynamic": "^2.4.8",
    "@angular/router": "^3.4.8",
    "@ng-bootstrap/ng-bootstrap": "^1.0.0-alpha.20",
    "@ngrx/core": "^1.2.0",
    "@ngrx/store": "^2.2.1",
    "angular-confirmation-popover": "^2.1.2",
    "angular2-moment": "^1.2.0",
    "bootstrap": "4.0.0-alpha.5",
    "core-js": "^2.4.1",
    "font-awesome": "^4.7.0",
    "ng2-cookies": "^1.0.4",
    "ng2-page-scroll": "^4.0.0-beta.4",
    "ng2-slim-loading-bar": "^2.4.0",
    "ng2-toastr": "^1.5.0",


"ngrx-store-localstorage" does not support aot!
