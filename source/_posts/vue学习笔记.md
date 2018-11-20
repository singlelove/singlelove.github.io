---
title: 'vue学习笔记'
date: 2018-07-05 17:20:04
tags: 学习
categories: 框架
---
## computed依赖mapState的值
```javascript
{
  computed: {
    paramA: () => 1,
    paramB: () => this.paramA
  }
}
{
  computed: {
    ...mapState({
      paramC
    }),
    paramD: () => this.paramC
  }
}
```
此时this.paramA可以正常获取到，但是this.paramC会报undefined，未仔细研究解决方式，可暂时通过method方式解决。

## computed和method中的this
1、computed和method中的this都自动绑定到了vue实例。
2、不要使用箭头函数，否则this将不会按照期望指向vue实例，而是undefined
3、在computed中使用箭头函数的话，也可以通过vm => vm.a的方式获取到vm实例

## 更新store无法触发component更新
* When you directly set an item with the index, e.g. vm.items[indexOfItem] = newValue
* When you modify the length of the array, e.g. vm.items.length = newLength
[参考官方文档](https://vuejs.org/v2/guide/list.html#Caveats)

## vuex
mutation必须是同步操作，action可以是异步操作，store只应该关注数据不应该有UI层面的变量

# to be continued



