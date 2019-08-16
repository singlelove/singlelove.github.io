---
title: react中mixin的危害
date: 2019-01-16 10:25:46
tags: 学习
categories: react
---
# 问题
## 隐式依赖
mixin可能引入不可见的方法、属性、状态，降低了代码的可读性和可维护性，当出现mixin的相互依赖和耦合时，这种情况更甚。

## 命名冲突
引入多个mixin可能导致方法、属性、状态的冲突

## 增加复杂性
由于 mixins 是侵入式的，它改变了原组件，所以修改 mixins 等于修改原组件，随着需求的增长 mixins 将变得复杂，导致滚雪球的复杂性。详见[mixins-cause-snowballing-complexity](https://reactjs.org/blog/2016/07/13/mixins-considered-harmful.html#mixins-cause-snowballing-complexity)

# 解决方案
## 性能优化场景
使用PureComponent替代PureRenderMixin

## 数据订阅场景
使用HOC替代mixin，比如下面的场景：
```jsx
var SubscriptionMixin = {
  getInitialState: function() {
    return {
      comments: DataSource.getComments()
    };
  },

  componentDidMount: function() {
    DataSource.addChangeListener(this.handleChange);
  },

  componentWillUnmount: function() {
    DataSource.removeChangeListener(this.handleChange);
  },

  handleChange: function() {
    this.setState({
      comments: DataSource.getComments()
    });
  }
};

var CommentList = React.createClass({
  mixins: [SubscriptionMixin],

  render: function() {
    // Reading comments from state managed by mixin.
    var comments = this.state.comments;
    return (
      <div>
        {comments.map(function(comment) {
          return <Comment comment={comment} key={comment.id} />
        })}
      </div>
    )
  }
});

module.exports = CommentList;
```
可以改写成

```jsx
function withSubscription(WrappedComponent) {
  return React.createClass({
    getInitialState: function() {
      return {
        comments: DataSource.getComments()
      };
    },

    componentDidMount: function() {
      DataSource.addChangeListener(this.handleChange);
    },

    componentWillUnmount: function() {
      DataSource.removeChangeListener(this.handleChange);
    },

    handleChange: function() {
      this.setState({
        comments: DataSource.getComments()
      });
    },

    render: function() {
      // Use JSX spread syntax to pass all props and state down automatically.
      return <WrappedComponent {...this.props} {...this.state} />;
    }
  });
}

// Optional change: convert CommentList to a function component
// because it doesn't use lifecycle methods or state.
function CommentList(props) {
  var comments = props.comments;
  return (
    <div>
      {comments.map(function(comment) {
        return <Comment comment={comment} key={comment.id} />
      })}
    </div>
  )
}

// Instead of declaring CommentListWithSubscription,
// we export the wrapped component right away.
module.exports = withSubscription(CommentList);
```
~~**个人理解**：HOC可以解决mixin上述问题中的后两点。~~
~~* 因为多个mixin共同作用时可能相互影响，而HOC是层层包裹的，不会出现名称污染；~~
~~* HOC不是侵入式的，没有改动原组件~~

HOC对比mixin的优势：
* 支持ES6 classes语法
* 解决了方法名可能重名导致覆盖的问题

HOC存在的缺陷：
* 依然不够直观。多个mixin共用时，不知道state是从哪个mixin引入的；而多个HOC共用时，也会存在不知道prop是从
哪个HOC传入的问题。
* 依然有重名问题。多个mixin共用时，state可能重名；多个HOC共用时，prop可能重名。

同时HOC还引入了新的问题：
* 需要添加displayName，方便调试，[参考](https://reactjs.org/docs/higher-order-components.html#convention-wrap-the-display-name-for-easy-debugging)
* 需要做props的透传，[参考](https://reactjs.org/docs/higher-order-components.html#convention-pass-unrelated-props-through-to-the-wrapped-component)
* 需要拷贝静态方法，[参考](https://reactjs.org/docs/higher-order-components.html#static-methods-must-be-copied-over)
* 是静态构建**关于这点还不是很理解**
* 包裹多个HOC时由于会产生无用组件，因此更容易造成"包装地狱"，如图：
![](/images/article/react中mixin的危害/wrapper hell.png) 

参考：
[使用 Render props 吧！](https://juejin.im/post/5a3087746fb9a0450c4963a5)
[Higher-order components vs Render Props](https://www.richardkotze.com/coding/hoc-vs-render-props-react)

更多HOC的介绍可阅读我的另一篇文章[react进阶](https://singlelove.github.io/2019/01/16/react%E8%BF%9B%E9%98%B6/#HOC%E4%BD%BF%E7%94%A8)

### 扩展阅读：
render props相比HOC的优势，[参考](https://www.richardkotze.com/coding/hoc-vs-render-props-react)
* 可以获得组件运行时的state和props，这一点HOC无法做到(**虽然还没想到具体的应用场景，不过我觉得这是render props相比HOC最大的优势**)
```jsx
class RenderProps extends React.Component {
  constructor(props) {
    super(props);
    this.state = {...};
  }
  render() {
    return (
      <WithMouse {..this.state} {...this.props}>    //通过这种方式可以获得组件运行时的state和props
        {mouse => (
          <WrapperComponent />
        )}          
    </WithMouse>
    )
  }
}
```
* 依赖更明确，知道state是由谁提供的
* 不存在方法或者state或者props的冲突
* 不需要额外处理静态方法，添加displayName，传递props等处理

不过render props同样带来了问题：
* shouldComponentUpdate/PureComponent使用时需要注意，需要采用[如下方式](https://reactjs.org/docs/render-props.html#be-careful-when-using-render-props-with-reactpurecomponent)定义render props，否则会失效引起冗余渲染
* 多个嵌套时会造成“callback hell”的代码，很不优雅，如下图

```jsx
// render props的方式
class RenderPropsComponent extends React.Component {
  render() {
    return (
      <WithHeader>
        {header => (
          <WithMouse>
          {mouse => (
            <WrapperComponent />
          )}          
        </WithMouse>
        )}
      </WithHeader>
    )
  }
}

// HOC的方式
const HOCComponent = withHeader(withMouse(WrapperComponent));
```

## 共用渲染
使用组件的形式替代
```jsx
var RowMixin = {
  // Called by components from render()
  renderHeader: function() {
    return (
      <div className='row-header'>
        <h1>
          {this.getHeaderText() /* Defined by components */}
        </h1>
      </div>
    );
  }
};

var UserRow = React.createClass({
  mixins: [RowMixin],

  // Called by RowMixin.renderHeader()
  getHeaderText: function() {
    return this.props.user.fullName;
  },

  render: function() {
    return (
      <div>
        {this.renderHeader() /* Defined by RowMixin */}
        <h2>{this.props.user.biography}</h2>
      </div>
    )
  }
});
```
可以改写成：
```jsx
function RowHeader(props) {
  return (
    <div className='row-header'>
      <h1>{props.text}</h1>
    </div>
  );
}

function UserRow(props) {
  return (
    <div>
      <RowHeader text={props.user.fullName} />
      <h2>{props.user.biography}</h2>
    </div>
  );
}
```

## 提供context
使用HOC替代mixin
```jsx
var RouterMixin = {
  contextTypes: {
    router: React.PropTypes.object.isRequired
  },

  // The mixin provides a method so that components
  // don't have to use the context API directly.
  push: function(path) {
    this.context.router.push(path)
  }
};

var Link = React.createClass({
  mixins: [RouterMixin],

  handleClick: function(e) {
    e.stopPropagation();

    // This method is defined in RouterMixin.
    this.push(this.props.to);
  },

  render: function() {
    return (
      <a onClick={this.handleClick}>
        {this.props.children}
      </a>
    );
  }
});

module.exports = Link;
```
可以改写成
```jsx
function withRouter(WrappedComponent) {
  return React.createClass({
    contextTypes: {
      router: React.PropTypes.object.isRequired
    },

    render: function() {
      // The wrapper component reads something from the context
      // and passes it down as a prop to the wrapped component.
      var router = this.context.router;
      return <WrappedComponent {...this.props} router={router} />;
    }
  });
};

var Link = React.createClass({
  handleClick: function(e) {
    e.stopPropagation();

    // The wrapped component uses props instead of context.
    this.props.router.push(this.props.to);
  },

  render: function() {
    return (
      <a onClick={this.handleClick}>
        {this.props.children}
      </a>
    );
  }
});

// Don't forget to wrap the component!
module.exports = withRouter(Link);
```

## 提供util方法
放到util.js里再引入
```jsx
var ColorMixin = {
  getLuminance(color) {
    var c = parseInt(color, 16);
    var r = (c & 0xFF0000) >> 16;
    var g = (c & 0x00FF00) >> 8;
    var b = (c & 0x0000FF);
    return (0.299 * r + 0.587 * g + 0.114 * b);
  }
};

var Button = React.createClass({
  mixins: [ColorMixin],

  render: function() {
    var theme = this.getLuminance(this.props.color) > 160 ? 'dark' : 'light';
    return (
      <div className={theme}>
        {this.props.children}
      </div>
    )
  }
});
```
可以改写成：
```jsx
var getLuminance = require('../utils/getLuminance');

var Button = React.createClass({
  render: function() {
    var theme = getLuminance(this.props.color) > 160 ? 'dark' : 'light';
    return (
      <div className={theme}>
        {this.props.children}
      </div>
    )
  }
});
```
