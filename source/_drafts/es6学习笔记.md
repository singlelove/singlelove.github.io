---
title: es6学习笔记
tags: 学习
categories: es6
---
# 函数作用域
es6规定函数本身的作用域，在其所在的块级作用域内，而es5会进行函数提升。示例中es6输出outside，es5输出inside
```javascript
function f() { 
    console.log('I am outside!'); 
}
(function () {
  if (false) {
    // 重复声明一次函数f
    function f() { 
        console.log('I am inside!'); 
    }
  }
  f();
}());
```
# 解构赋值常用用途
- 交换变量的值。如[x,y] = [y,x]；
- 提取json数据。如 let {id,name} = jsonData;