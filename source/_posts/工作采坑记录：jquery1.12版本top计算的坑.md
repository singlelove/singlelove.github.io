---
title: 工作采坑记录：jquery1.12版本top计算的坑
date: 2018-04-17 16:32:03
tags: 问题记录
categories: jquery
---
# 前言
使用[jquery chosen](https://github.com/harvesthq/chosen)插件的时候遇到了个奇怪的问题，在用鼠标滚轮滑动下拉菜单过程中，会突然跳转到下拉菜单最底部，往上回滚也无济于事，会再次跳转到底部，但是看了官网插件并无此问题，于是开始debug。
# scroll事件覆盖
首先怀疑是本地代码对scroll事件进行了覆盖导致的冲突，但是检查完之后发现并无异常行为。
# 插件highlight事件断点调试
仔细观察bug后发现滚动时行为正常，但是当鼠标移到下拉菜单上，高亮选中项时，就会发生异常跳转，于是怀疑是插件highlight操作时出现异常。
~~~javascript
    Chosen.prototype.result_do_highlight = function(el) {
      var high_bottom, high_top, maxHeight, visible_bottom, visible_top;
      if (el.length) {
        this.result_clear_highlight();
        this.result_highlight = el;
        this.result_highlight.addClass("highlighted");
        maxHeight = parseInt(this.search_results.css("maxHeight"), 10);
        visible_top = this.search_results.scrollTop();
        visible_bottom = maxHeight + visible_top;
        high_top = this.result_highlight.position().top + this.search_results.scrollTop();
        high_bottom = high_top + this.result_highlight.outerHeight();
        if (high_bottom >= visible_bottom) {
          return this.search_results.scrollTop((high_bottom - maxHeight) > 0 ? high_bottom - maxHeight : 0);
        } else if (high_top < visible_top) {
          return this.search_results.scrollTop(high_top);
        }
      }
    };
~~~
上面的代码是插件执行highlight的部分，对比官网例子之后发现当下拉菜单往下滚动时，本地***this.result_highlight.position().top***的值异常大，官网例子的对应参数取值在[0, 240]之间，也就是下拉菜单的可视区域的最大高度，于是确定是***this.result_highlight.position().top***的计算有问题。
# css样式
为什么本地***this.result_highlight.position().top***的值会出错呢，[jquery](https://api.jquery.com/position/)官网说明比较简略，position计算的是当前元素与父元素的相对距离。我首先想到的是会不会是本地的css样式影响了position的计算，于是把插件移到了最顶端，注释掉了几乎所有css，只保留了插件的样式，但是bug依然存在。
# issues里找答案
万般debug无果，我又把目光投向了插件在github上的issues，看是否有其他人遇到同样的问题，果然找到了一个[issue](https://github.com/harvesthq/chosen/issues/2506)，也找到了解决方案
![](/images/article/工作采坑记录：jquery1.12版本top计算的坑/workaround.png) 
试了一下，果然解决了。那为什么会产生这个问题呢，继续看issues下的评论，发现了这是jquery1.12中的一个[bug](https://github.com/jquery/jquery/commit/49833f7795d665ff1d543c4f71f29fca95b567e9)。在jquery1.12和2.2中top的计算出现了bug，把scrollTop的值也算进去了。
# 总结
总结了下，在这个bug的定位解决中还是走了不少弯路的。如果一开始就注意到官网中要求的插件依赖jQuery support: 1.7+，就能更早的定位到问题。总结了下插件bug的定位过程，感觉按照下面的过程去定位会比较好：
* 1、看本地是否满足了插件依赖的版本要求
* 2、看issues里是否有别人遇到了同样的问题
* 3、最后再断点插件相关部分的代码自己找原因
