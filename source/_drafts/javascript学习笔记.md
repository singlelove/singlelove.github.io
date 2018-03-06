---
title: javascript学习笔记
tags: 学习
categories: javascript
---
# 闭包
有权访问定义在另一个函数作用域中的变量的函数。因为定义在闭包中的函数可以“记忆”它创建时候的环境。
示例：
```javascript
var method = function (){
    var param = 'a';    //param就是method的上下文环境
    return function display(){
        ...    //此处才是函数的主体
    }
}
```
其中dispaly就是一个闭包；

# 模块
一个定义了私有变量和函数的函数；利用闭包创建可以访问私有变量和函数的特权函数；最后返回这个特权函数或者把他们保存到一个可以访问的地方。

# 级联
为那些没有返回值的方法增加一个return this可以启用级联，即可以使用func1().func2()的形式依次调用同一个对象的多个方法。

# 柯里化：把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数。

# 全局变量
省略var定义的变量是全局变量，但是在严格模式下会抛出ReferenceError错误。

# typeof
typeof null也会返回“object”，因为null是一个空的对象引用；对未初始化的变量和未声明的变量使用typeof均返回undefined。

# 函数声明与函数表达式
- 函数声明：function method(){...}会被解析器提升到代码顶部，因此先有method的调用，再有method的声明不会有错；
- 函数表达式：var method = function(){...};并不会被提升到代码顶部，因此运行时会报错，详见《javascript高级程序设计》p112；

# arguments.callee
arguments.callee是一个指向函数的指针(严格模式下，访问它会报错)，通常用来消除函数的执行与函数名之间的耦合，详见《javascript高级程序设计》p114；

# caller
函数对象中还有一个属性caller，其值为调用当前函数的函数的引用(caller是只读的，无法赋值)，详见《javascript高级程序设计》p115；

# 错误处理
##服务器端错误记录
使用Image对象发送请求更灵活，详见《javascript高级程序设计》p512；
 
# 最佳实践
- 减少与null的比较，尽量用期望的类型进行判断，如typeof(parma) === "string"；基本类型用typeof，特定对象比如array用instanceof（注：在多框架中instanceof不一定适用，详见《javascript高级程序设计》p665）
- 必须要对dom进行更新时，考虑使用文档片段创建dom结构，在所有操作都执行完成之后，再加文档片段添加到现存的文档当中，保证只有一次的现场更新，详见《javascript高级程序设计》p674；
- innerHtml比createElement和appendChild方法效率更高

# 匿名函数
[参考](https://www.zhihu.com/question/20249179)
- 类似函数声明形式的都是错误写法，会报语法错误，如function(){}()和function f(){}()
- 函数表达式形式的写法才是匿名函数，如var a = function(){}()，！function(){}()等（一元运算符!把后者变成了函数表达式）
- function f(){}(expression)不报语法错误，但不是立即执行函数，只是一个函数声明与一个(expression)函数表达
  
# merge、assign、extend区别
[参考](https://scarletsky.github.io/2016/04/02/assign-vs-extend-vs-merge-in-lodash/)
- merge是递归拷贝，assign/extend是浅拷贝，如果属性本身是对象，则直接拷贝整个对象，如图
{% qnimg article/javascript学习笔记/copy.png %}
- assign不会拷贝原型链上的属性，而extend会。

# javascript的事件驱动机制
[参考](https://juejin.im/post/59e21e8551882578db27c364)
