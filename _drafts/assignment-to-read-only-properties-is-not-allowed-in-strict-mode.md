---
layout: post
title: Assignment to read-only properites is not allowed in strict mode
categories: angular template strict-mode
---
# 场景

在AngularJS 2 template中使用...[style]=\"safeStyle\"...时，MSEdge报错...

> Assignment to read-only properites is not allowed in strict mode

# 整个

当红色的错误在控制台里跳动，不免又怀疑是Edge/IE系列的错，一番探究之后，剧情反转，再次警醒自己——“一切都是你的错”。


{% highlight typescript linenos %}
@Component({
  selector: 'read-more',
  template: `
    <div [innerHTML]="currentText" [style]="safeStyle" [style.white-space]="isCollapsed ? '' : 'pre-wrap'">
    </div>
    `
})
...
{% endhighlight %}

{% highlight typescript linenos %}
@Component({
  selector: 'read-more',
  template: `
    <div [innerHTML]="currentText" [attr.style]="safeStyle" [style.white-space]="isCollapsed ? '' : 'pre-wrap'">
    </div>
    `
})
...
{% endhighlight %}


{% highlight javascript linenos %}
{% endhighlight %}
