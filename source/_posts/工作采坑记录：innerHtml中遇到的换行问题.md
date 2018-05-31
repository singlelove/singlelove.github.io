---
title: '工作采坑记录: innerHtml中遇到的换行问题'
date: 2018-05-31 09:32:23
tags: 问题记录
categories: w3c
---
# 前言
项目里遇到了一个奇怪的bug，有个通过[jquery-confirm](https://craftpip.github.io/jquery-confirm/)生成的弹窗，弹窗内容由
[handlebar](https://handlebarsjs.com/)生成，其中有一个textarea文本输入框。当在textarea先输入一个换行，之后内容随意，然后
点击保存关闭弹窗。下次再重新打开的弹窗的时候，开头的换行一定会丢失(之后的换行都能正常保存，多个换行开头也只丢失一个)。

# debug过程
首先检查了接口返回的数据，正常。
接着检查了handlebar返回的html模板字符串，正常，包含了正确数量的换行。
最后只剩下jquery-confirm了，于是通过chrome的开发者工具继续单步debug。
最终确认是的jquery-confirm中，通过ele.innerHtml将handlebar生成的html模板字符串塞入弹窗dom的过程中，造成了开头换行符的丢失。
赋值前换行符还存在
{% qnimg 'article/工作采坑记录：innerHtml中遇到的换行问题/linebreakExist.png' %}
赋值后换行符已丢失
{% qnimg 'article/工作采坑记录：innerHtml中遇到的换行问题/linebreakMissing.png' %}
至此，确认是innerHtml的锅。

# workaround
不过在网上搜索了innerHtml相关的信息，并没有找到关于这个bug的解决，甚至连这个bug都没人提到，应该是我搜索的方式不对orz。
于是只能暂时用了一个workaround的方式，在展示弹窗内容的时候，如果发现是以换行开头的，就加一个换行用来抵消innerHtml的影响，
之后有空再研究研究有没有更优雅的解决方式。
```javascript
content = content.replace(/^\r\n/,'\r\n\r\n');
```

...to be continued
