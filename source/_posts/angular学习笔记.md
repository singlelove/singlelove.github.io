---
title: angular学习笔记
date: 2018-03-06 16:58:58
tags: 学习
categories: angular
---
- 引用图片模板时，一定要用ng-src替代src，不然src会直接使用&#123;&#123;&#125;&#125;作为图片地址，并不会去解析。
- 内容的更新由控制器负责。不同区域有不同模板的话，一种方式是使用ng-show/ng-hide，即有内容了才显示出来，第二种方式是使用ng-include，加载不同模板，第三种方式是使用ng-switch，类似tab标签效果
- ngrepeat默认不允许有重复的数据，可以通过加track by $index解决
![](/images/article/angular学习笔记/ngrepeat.png) 
- ng-repeat下每一个子代都有一个scope，因此要用$root（验证成功，模板里用$root，控制器里用$rootscope）或者$parent（未验证成功）定义才能在外层访问。
![](/images/article/angular学习笔记/ngrepeatScope.png) 
- Angular异步编程用"$q"，defer对象的声明及promise对象的返回放在异步操作外层
![](/images/article/angular学习笔记/defer.png) 
- 双向绑定“失效”的原因
  - $scope作用域的不同造成的
  - 绑定的变量在第三方插件(或者监听器中发生改变)，则需要调用$scope.$apply()手动更新视图。
- 关于如何在angular完全加载，即各种ng指令编译链接完成，生成了动态的dom组件之后执行某些操作：自定义directive，并在link里添加$timeout，把要执行的操作放到$timeout函数里，delay可设置为0。[未找到满意方法](https://github.com/angular/angular.js/issues/734)
![](/images/article/angular学习笔记/afterRender.png) 
- 尽量不要直接用&#123;&#123;&#125;&#125;的方式绑定文本，而用ng-bind，否则在加载缓慢时，浏览器会直接显示&#123;&#123;&#125;&#125;，用户体验不好
- Angular内置过滤器filter
  - currency: currency允许我们设置自己的货币符号，默认情况下会采用客户端所处区域的货币符号。可以这样使用：&#123;&#123; 3600 | currency: "$￥"&#125;&#125;返回结果为$￥123.00
  - number: number过滤器将数字格式化成文本，它的参数是可选的，用来控制小数点后的截取位数如果传入的是一个非数字字符，会返回空字符串  可以这样使用：&#123;&#123; 3600 | number:2&#125;&#125;返回结果为：3,600.00
  - lowercase: lowercase将字符串转换为小写可以这样使用：&#123;&#123; "HEllo" | lowercase&#125;&#125;返回结果为：hello
  - uppercase: uppercase将字符串转换为大写可以这样使用：&#123;&#123; "HEllo" | uppercase&#125;&#125;返回结果为：HELLO
  - json: json过滤器可以将一个JSON或者JavaScript对象转换成字符串。这个过滤器对调试相当有用可以这样使用：&#123;&#123; &#123;"name":"dreamapple","language":"AngularJS"&#125; | json&#125;&#125;返回结果为：&#123; "name": "dreamapple", "language": "AngularJS" &#125;
  - date: date过滤器将日期过滤成你想要的格式，这个实在是很好的过滤器。这个过滤器用法很多我这里列举几种常用的&#123;&#123; today | date: "yyyy - mm - dd"&#125;&#125;结果为：2015 - 15 - 13&#123;&#123; today | date: "yyyy - mm - dd HH:mm::ss"&#125;&#125;    结果为：2015 - 18 - 13 20:18::38
- directive中的特殊符号
  - scope会为指令创建独立的scope，左边为属性或函数名，右边为绑定属性或函数，可用修饰符罗列如下：
    - '='：绑定标签中的属性所指定的$scope上的属性；
    - '&'：绑定标签中的属性所指定的$scope上的属性，该属性必须为函数回调；
    - '@'：绑定标签上的属性所指定的html字面量/直接值；
    ![](/images/article/angular学习笔记/directive-1.png) 
    '='表示指令自身的独立scope中$scope.customer对应的标签属性值为info
    ![](/images/article/angular学习笔记/directive-2.png) 
    ![](/images/article/angular学习笔记/directive-3.png) 
    '&'表示将标签属性中的onClose函数绑定当指令自身独立scope中的$scope\.close属性上  
    ![](/images/article/angular学习笔记/directive-4.png) 
  - Require表示指令依赖的controller，可用修饰符如下
    - '^'前缀会使$compile在其父元素上搜索，找不到会抛出错误；
    - '?'前缀在当前指令寻找，找不到会将null作为第四个参数传入link；
    - 无前缀则只在当前指令寻找，找不到会抛出错误；
- 动态生成的directive，若其scope不为空，则生成的directive的scope的父scope是用于生成directive的directive的scope
![](/images/article/angular学习笔记/svnProjectItem.png) 
![](/images/article/angular学习笔记/svnAddItem.png) 
如图，普通svnProjectItem的父scope是A，svnAddItem的scope为B，通过svnAddItem生成的svnProjectItem的父scope是svnAddItem的scope即B，因此需要在svnAddItem中再指向A中的方法，才能使svnProjectItem指令中的delete方法正确调用到A中的对应方法
- 如果要将directive A中的方法暴露在其他directive B时，将方法写在A指令的controller中，在B指令中通过require: ‘A’的形式引入A的controller，从而在B中完成调用。
- Fis3构建时，angular的依赖注入一定要采用标准的写法，否则会出现压缩后的代码无法正确注入依赖的情况



