---
title: react中创建组件的几种方式
date: 2019-01-22 11:10:06
tags: 学习
categories: react
---
# 三种创建方式
## createClass
这是react在ES5时期使用的创建组建的方式
```jsx
var React = require("react");
var Greeting = React.createClass({
  
  propTypes: {
    name: React.PropTypes.string //属性校验
  },

  getDefaultProps: function() {
    return {
      name: 'Mary' //默认属性值
    };
  },
  
  getInitialState: function() {
    return {count: this.props.initialCount}; //初始化state
  },
  
  handleClick: function() {
    //用户点击事件的处理函数
  },

  render: function() {
    return <h1>Hello, {this.props.name}</h1>;
  }
});
module.exports = Greeting;
```
在createClass中，React对属性中的所有函数都进行了this绑定，也就是如上面的hanleClick其实相当于handleClick.bind(this) 。

## class component
随着ES6的到来，带来了类和继承语法级别的支持，所以在ES6中，React推荐使用class的方式创建组件
```jsx
import React from 'react';
class Greeting extends React.Component {

  constructor(props) {
    super(props);
    this.state = {count: props.initialCount};
    this.handleClick = this.handleClick.bind(this);
  }
  
  //static defaultProps = {
  //  name: 'Mary'  //定义defaultprops的另一种方式
  //}
  
  //static propTypes = {
    //name: React.PropTypes.string
  //}
  
  handleClick() {
    //点击事件的处理函数
  }
  
  //handleClick = () => {
  //}
  
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}

Greeting.propTypes = {
  name: React.PropTypes.string
};

Greeting.defaultProps = {
  name: 'Mary'
};
export default Greating;
```
而在面向对象的语法中类的属性通常被称作静态(static)属性，这也是为什么props还可以像上面注释掉的方式来定义。
用这种方式创建组件时，React并没有对内部的函数，进行this绑定，所以如果你想让函数在回调中保持正确的this，就要手动对需要的函数进行this绑定，有两种方式：
* 构造函数中使用this.handleClick = this.handleClick.bind(this);
* 函数声明时采用箭头函数，如注释中的handleClick = () => { return ...}

## functional component
上面我们提到的创建组件的方式，都是用来创建包含状态和用户交互的复杂组件，当组件本身只是用来展示，所有数据都是通过props传入的时候，我们便可以使用Stateless Functional Component来快速创建组件。例如下面代码所示:
```jsx
import React from 'react';
const Button = ({
  day,
  increment
}) => {
  return (
    <div>
      <button onClick={increment}>Today is {day}</button>
    </div>
  )
}

Button.propTypes = {
  day: PropTypes.string.isRequired,
  increment: PropTypes.func.isRequired,
}
```
这种组件，没有自身的状态，相同的props输入，必然会获得完全相同的组件展示。因为不需要关心组件的一些生命周期函数和渲染的钩子，所以不用继承自Component显得更简洁。

# 比较
class component与createClass本质上没什么区别，只是一个支持ES6，一个不支持
functional component与class component相比，主要区别在于function component是无状态的，也没有生命周期函数，但是带来了以下几个优势：
* 没有了this的困扰，在class component中事件回调经常需要通过bind或者箭头函数绑定this
* 强制视图、逻辑分离，耦合更低。functional component是无状态组件，适合用于表现类组件的创建。
* 可读性、可测试性都得到提高。因为functional component是纯函数，指定input即可确定output
* 潜在的性能提升。虽然目前functional component相比class component并没有性能提升，但是React官方计划在未来为functional component做优化，避免不必要的检查和内存分配，比如[这篇文章](https://medium.com/missive-app/45-faster-react-functional-components-now-3509a668e69f)指出，通过合理的优化，functional component可以实现45%的性能提升

很多时候我们创建了functional component，但是后来又需要为组件添加一些状态或者context或者需要调用生命周期函数，此时不必把functional component改写成class component，使用[react hooks](https://reactjs.org/docs/hooks-overview.html)或者[recompose](https://github.com/acdlite/recompose)即可。
这里更推荐使用官方的react hooks，它不仅为functional component带来了使用状态、context、生命周期函数的能力，使其可以完成class component的功能，还避免了“包装地狱”的引入
> “包装地狱”是目前react中很常见的一个问题，在使用context或者HOC或者render props的时候因为引入了无用的标签，很容易就产生了很深的标签嵌套。

# 参考
[谈一谈创建React Component的几种方式](https://segmentfault.com/a/1190000008402834)
[React Functional or Class Components: Everything you need to know](https://programmingwithmosh.com/react/react-functional-components/)
[前端架构杂思录：议 Function Component 与 Hooks](http://taobaofed.org/blog/2018/11/27/hooks-and-function-component/)