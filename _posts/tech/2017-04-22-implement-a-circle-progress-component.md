
我需要一个环装进度条来显示百分比进度，做了以下这些尝试：

* [`ngx-charts`](https://github.com/swimlane/ngx-charts): 文档和API尚可，但细节总不够让人满意，反复尝试，最后放弃，感觉这个项目有很多细节还没来得及优化，可能要再等等。
* [`ng2-charts`](https://github.com/valor-software/ng2-charts): 我在用`valor-soft`的其他组件，体验不错，可是这个组件没法拼出环状进度条。
* [`angular2-highcharts`](https://github.com/gevgeny/angular2-highcharts): [highcharts](http://www.highcharts.com/)我用过，功能、文档和API都很棒，但`angular2-highcharts`也没有提供环状百分比进度条，我试着用甜甜圈来模拟，效果不满意。
* [`angular2-google-chart`](https://github.com/vimalavinisha/angular2-google-chart): [`Google Chart Gauge`](https://google-developers.appspot.com/chart/interactive/docs/gallery/gauge) 经过参数调整，模拟出效果不错的环状进度条，而且这个组件库质量最好，各方面都占优，可，这宝贝不提供离线代码，心思费了不老少，结果还是用不了。

写这篇文章的时间，距离这些失败的尝试已经两周，细节回忆不准了。当时确实应该用文字记录这些细节，仅仅两周就忘得像没有做过这件事。

再后来，我找到了[`angular2-circle-progress`](https://github.com/Feridum/angular2-circle-progress/)，觉得自己做一个也不算很麻烦，于是决定自己做。

## 工作结果

陆续花了两周，发布一个基本可用的版本。

* [ng-circle-progress NPM包](https://www.npmjs.com/package/ng-circle-progress)
* [ng-circle-progress Github项目](https://github.com/bootsoon/ng-circle-progress)
* [组件演示](https://bootsoon.github.io/ng-circle-progress/)

[![Demo](/assets/tech/images/2017/04/22/example.svg)](https://www.npmjs.com/package/ng-circle-progress)


## 回顾

> 记忆是思考的残留。

即便最终代码被发布出来时，看上去清晰也完整，但产生完整且清晰的代码的过程却不是这样，而是恰恰相反，几乎是零敲碎打。

所以即便是自己写出的代码，也没法再现自己的思考过程，所以我要用文字记录思维过程：遇到了哪些困难？我是怎么思考并最终解决的？哪些方面做得不够好，还需要改进吗？

为了记住，最直接的方式是记录。

为了做成这件小事，我需要解决几个问题：

1. 确定基本功能和造型；
2. 学习SVG画弧指令；
3. 计算圆弧坐标的方法；
4. 编译出Angular Module；
5. 发布npm包

## 确定基本功能和造型

[谷歌关键字`circle progress`](https://www.google.com/search?q=circle+progress&tbm=isch)，总结图片搜索结果，大概有以下几种样式：

![](/assets/tech/images/2017/04/22/example-a.png)

![](/assets/tech/images/2017/04/22/example-b.png)

![](/assets/tech/images/2017/04/22/example-c.png)

![](/assets/tech/images/2017/04/22/example-d.png)

我想要的一个安静的“百搭款”组件，最后，我选择了最后一种，而且，我觉得最终也许能够通过参数和事件实现更精细地控制，比如动态改变颜色等。

## Arc指令怎么确定弧的形状？

SVG的Arc指令使用四组参数确定弧的形状：起点、终点、半径、标志位。

* 起点和终点：当圆环进度条的进度达到100%时，终点和起点重合。我没法告诉Arc指令，让它从某点开始，画一个360度的圆弧，然后又回到这个点，这是不行的。变通的办法有两个，一是画两个半圆弧拼在一起，二是把终点稍做偏移，小数点后两位的偏移即可，显然后者更简单。

* 半径：我要画圆弧，x-radius和y-radius相等。

* 标志位：这两个标志位不易理解，[MDN SVG Tutorial](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Paths#Arcs) 有个案例，但使用了闭合图形，反倒增加了理解难度。我写了个小SVG，可以更直观地理解它们:

  ![](/assets/tech/images/2017/04/22/large-arc-flag-and-sweep-flag.svg)

  1. large-arc-flag: 当sweep-flag确定时，有两条弧满足条件，一条长，一条短。“0”是短弧，“1”是长弧。
  2. sweep-flag: 当large-arc-flag确定时，有两条弧满足条件，一条逆时针，一条顺时针。“0”是逆时针，“1”是顺时针。

  不管什么情况下，我都选择逆时针画弧。
  
  当百分比进度不足50%，起点S、圆心O和终点E形成的角&angle;SOE&lt;180&deg;，选短弧。
  
  当百分比进度超过50%，&angle;SOE&gt;180&deg;，选长弧。

  进度等于50%时，长弧和短弧重合。
  

## 根据百分比计算圆弧终点

我不准备接受弧的起始角度和终止角度作为参数，而是从0度开始画弧到360度，把完整的圆弧当做100%进度。

所以把圆周上位于圆心正上方的点确定为弧的起点，使用polarToCartesian函数，输入圆心坐标、半径和旋转角度，得到弧的终点坐标。

{% highlight typescript %}

function polarToCartesian = (centerX, centerY, radius, angleInDegrees) => {
    let angleInRadius = angleInDegrees * Math.PI / 180;
    let x = centerX + Math.sin(angleInRadius) * radius;
    let y = centerY - Math.cos(angleInRadius) * radius;
    return { x: x, y: y }
}

let endPoint = polarToCartesian(centre.x, centre.y, radius, 360 * percentage / 100);

{% endhighlight %}

## [Yeoman generator angular2-library](https://github.com/jvandemo/generator-angular2-library)

完成了核心功能，我想把功能封装成 NgModule 并用NPM模块的形式发布，以便使用。

我知道，从头手写一个符合标准的模块，是多么繁琐的事情，没有趁手的工具，那怎么行。

通过StackOverflow找到[Yeoman generator angular2-library](https://github.com/jvandemo/generator-angular2-library)：

> If you want to create an Angular library with directives, services and/or pipes, then this generator is just what you need.
>
> This generator aligns with the [official Angular Package Format](https://goo.gl/AMOU5G) and automatically generates a [Flat ES Module](http://angularjs.blogspot.com/2017/03/angular-400-now-available.html), a single metadata.json and type definitions to make your library ready for AOT compilation by the consuming Angular application.
> 
> Watch [Jason Aden's talk](https://www.youtube.com/watch?v=unICbsPGFIA) to learn more about the Angular Package Format.

这段简介清晰明了，很有说服力，我马上下载来试用了一下，也感觉挺顺手的。

（这段简介里面有三个扩展相关知识的链接，我竟然都没跟下去哦，写到这里，觉得自己很无耻，既然写模块写组件，竟然忽视相关的底层知识，长此以往，怎么对得起自己，想把自己饿死吗？明天必须学完这三个链接。）

## 允许 `NgCircleProgressModule.forRoot(configuration)` 配置全局默认值

封装成模块时，要解决一个问题：

* 允许模块使用者通过`NgCircleProgressModule.forRoot({radius:90}})`来设置“用户默认值”
* 接受模板中指定的`<circle-progress [progress]="75" [radius]="100"></circle-progress>`来设置具体的组件的参数

{% highlight typescript %}

//在模块声名中为DI指定CircleProgressOptions的provider使用调用者forRoot中传入的options
@NgModule({
  imports: [
    CommonModule
  ],
  declarations: [
    CircleProgressComponent,
  ],
  exports: [
    CircleProgressComponent,
  ]
})
export class NgCircleProgressModule {
  static forRoot(options: CircleProgressOptionsInterface = {}): ModuleWithProviders {
    return {
      ngModule: NgCircleProgressModule,
      providers: [
        {provide: CircleProgressOptions, useValue: options}
      ]
    };
  }
}

//DI为CircleProgressComponent的constructor传入的CircleProgressOptions就是用户在forRoot是传入的options
export class CircleProgressComponent implements OnChanges {

  @Output() onClick: EventEmitter<any> = new EventEmitter();

  @Input() class: string;
  @Input() backgroundColor: string;
  @Input() backgroundOpacity: number;

  @Input() radius: number;
  @Input() space: number;
  @Input() percent: number;
  @Input() toFixed: number;
  @Input() maxPercent: number;
  @Input() renderOnClick: boolean;

  //...

  options: CircleProgressOptions = new CircleProgressOptions();

  constructor(defaultOptions: CircleProgressOptions) {
    Object.assign(this.options, defaultOptions);
  }

  private applyOptions = () => {
    for (let name of Object.keys(this.options)) {
      if (this.hasOwnProperty(name) && this[name] !== undefined) {
        this.options[name] = this[name];
      }
    }
  }
  
  render = () => {
    this.applyOptions();
    this.draw();
  }

{% endhighlight %}



## 编译得到Angular Module