---
title: CSS学习笔记
date: 2018-03-06 10:02:53
tags: 学习
categories: css
---

# 页内跳转anchor link
- 方式一：&lt;a&gt;标签href="#targetName"，目标元素必须为&lt;a&gt;标签，然后name="targetName"
- 方式二：&lt;a&gt;标签href="#targetId"，目标元素id="targetId"
> angular中要做修改，跳转发起元素中绑定事件，目标元素id="targetId"，事件执行以下操作（注意引入\$location,和\$anchorScroll）
\$location.hash("targetId");
\$anchorScroll();

# &lt;a&gt;只用作get请求
&lt;a&gt;(用作按钮)应该只用于get请求，不要用post请求，即不能用来更新服务器，因为可能出现意外结果，如内容消失。详见《精通css》p92

# rem
rem：相对于html元素的大小，为了便于计算，可设置html{font-size:62.5%}，这样1rem = 10px

# 清理浮动常用方法
- 添加一个进行清理的元素，缺点：增加了不必要的标记
- 让父元素浮动，使用某个元素(如站点页脚)对它进行清理，缺点：增加代码复杂度
- 使用overflow:hidden，缺点：不适用所用情况，因为会截断内容`(推荐)`

# float与inline-block
- 需要文字环绕图片效果时推荐float
- 其他情况个人倾向于用inline-block
- inline-block元素间会产生空白，推荐的解决方案是：在父元素中设置font-size:0,用来兼容chrome，而使用letter-space:-N px（N需要根据不同字号作调整）来兼容safari:

# 外边距合并
两个或多个毗邻（父子元素或兄弟元素）的普通流中的块元素垂直方向上的 margin 会发生叠加。
> 其中毗邻是指没有被非空内容、padding、border 或 clear 分隔开。

# animation
- 对元素A使用rotate3d(x,y,z,deg)之后，需要在其父元素B添加`-webkit-perspective`(暂时没有浏览器直接支持perspective)属性(透视距离)才能使A元素看起来具有3D效果。
- 问题1中要使A元素的子元素C也具有3D透视效果，则需要在A元素上也添加`-webkit-perspective`属性。
- 对一帧动画执行旋转、位移复合操作时，采用如下写法：`transform: rotateX(45deg) translateX(10px)`