---
title: react进阶
date: 2019-01-16 10:59:19
tags: 学习
categories: react
---
# 性能优化
## 使用PureComponent替代Component
[参考](https://reactjs.org/docs/react-api.html#reactpurecomponent)
使用PureComponent时，当state或者props不发生改变时，不会导致重新render。
注意PureComponent的比较是浅比较，当数组或对象的元素改变时，也不会引起render，
可采用如下方法实现重新render：
* 调用forceUpdate()方法
* 使用immutable objects
* 使用assign构造新对象或者slice构造新数组

## 不要将索引作为key
react做diff算法重新渲染的时候，会比较更新前、更新后的key值，如果发生变化就重新渲染。
而索引作为key时，一旦数组中插入或删除元素，很容易引起索引的变化，会导致很多不必要的重新渲染。
[参考](https://react.css88.com/docs/reconciliation.html#keys)

# HOC使用
HOC的两种主要实现方式：
* 属性代理。 高阶组件通过被包裹的React组件来操作props
* 反向继承。 高阶组件继承于被包裹的React组件

```jsx
//属性代理写法
import React from 'react';

const ppHoc = WrappedComponent => class extends Component {
    render() {
      return <WrappedComponent {...this.props}/>
    }
}

export default ppHoc;
```
```jsx
//反向继承写法
import React from 'react';

const iiHoc = WrappedComponent => class extends WrappedComponent {
    render() {
      console.log(this.state, 'state');
      return super.render();
    }
}

export default iiHoc;
```

## 属性代理
* 控制props

```jsx
import React, { Component } from 'react';

const propsProxyHoc = WrappedComponent => class extends Component {

  handleClick() {
    console.log('click');
  }

  render() {
    return (<WrappedComponent
      {...this.props}
      handleClick={this.handleClick}
    />);
  }
};
export default propsProxyHoc;
```
* 通过refs使用引用(**官方不建议过度依赖refs**)

```jsx
import React, { Component } from 'react';

const refHoc = WrappedComponent => class extends Component {

  componentDidMount() {
    console.log(this.instanceComponent, 'instanceComponent');
  }

  render() {
    return (<WrappedComponent
      {...this.props}
      ref={instanceComponent => this.instanceComponent = instanceComponent}
    />);
  }
};

export default refHoc;
```
* 抽象state，比如把不受控组件变成受控组件

```jsx
// 普通组件Login
import React, { Component } from 'react';
import formCreate from './form-create';
  
@formCreate
export default class Login extends Component {
  render() {
    return (
      <div>
        <div>
          <label id="username">
            账户
          </label>
          <input name="username" {...this.props.getField('username')}/>
        </div>
        <div>
          <label id="password">
            密码
          </label>
          <input name="password" {...this.props.getField('password')}/>
        </div>
        <div onClick={this.props.handleSubmit}>提交</div>
        <div>other content</div>
      </div>
    )
  }
}
```
展示与逻辑耦合，不利于复用，如果用HOC的方式实现：
```jsx
//HOC
import React, { Component } from 'react';

const formCreate = WrappedComponent => class extends Component {

  constructor() {
    super();
    this.state = {
      fields: {},
    }
  }

  onChange = key => e => {
    this.setState({
      fields: {
        ...this.state.fields,
        [key]: e.target.value,
      }
    })
  }

  handleSubmit = () => {
    console.log(this.state.fields);
  }

  getField = fieldName => {
    return {
      onChange: this.onChange(fieldName),
    }
  }

  render() {
    const props = {
      ...this.props,
      handleSubmit: this.handleSubmit,
      getField: this.getField,
    }

    return (<WrappedComponent
      {...props}
    />);
  }
};
export default formCreate;
```
* 其他元素包裹，方便布局(**感觉这个用父组件实现更合适**)

## 反向继承
* 渲染劫持

```jsx
//hijack-hoc
import React from 'react';

const hijackRenderHoc = config => WrappedComponent => class extends WrappedComponent {
  render() {
    const { style = {} } = config;
    const elementsTree = super.render();
    console.log(elementsTree, 'elementsTree');
    if (config.type === 'add-style') {
      return <div style={{...style}}>
        {elementsTree}
      </div>;
    }
    return elementsTree;
  }
};

export default hijackRenderHoc;
```
* 控制state

## 与父组件的区别
* HOC是统一功能的抽象，强调逻辑与UI分离。因此与UI相关的，建议使用父组件实现；
与UI不相关的，如校验、权限、请求发送、数据转换这类，通过数据变化间接控制DOM，可以使用HOC抽象
* HOC的解耦性比父组件更高，参考[此文](https://github.com/sunyongjian/blog/issues/25)应用场景第三点
关于添加样式和添加处理函数的container的解释

## 使用上的注意点(约束)
参考[react进阶之高阶组件](https://github.com/sunyongjian/blog/issues/25)

参考：
* [react进阶之高阶组件](https://github.com/sunyongjian/blog/issues/25)
* [精读 React 高阶组件](https://toutiao.io/posts/jrv8l8/preview)
* [深入理解 React 高阶组件](https://www.css88.com/archives/9462)
