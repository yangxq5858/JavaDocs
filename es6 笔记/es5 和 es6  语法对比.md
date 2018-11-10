# ES5与ES6对比

## 1. 模块引用

   1.在ES5里，引入React包基本通过require进行，代码类似这样：

```
// ES5
var React = require('react');
var { Component, PropTypes } = React;
console.log(Component);
```

在ES6里，import写法更为标准.

```
// ES6
import React, { Component, PropTypes } from 'react';
console.log(Component);
```

## 2. 导出单个类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 在ES5里面，要导出一个类别的模块用，一般通过module.exports来导出。
var React = require('react');
var MyComponents = React.createClass({
   render: function() {
    return (<div>1111</div>)
   }
});
module.exports = MyComponents;

// 引用如下：
var c1 = require('./MyComponents');

// 在ES6里面 通过export default来导出类
// es6
export default class MyComponent extends Component {
  render() {

  }
}
// 引用如下：
import MyComponent from './MyComponent';
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

## 3. 定义组件

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// ES5里面， 通过React.createClass来定义一个组件类
// es5
var React = require('react');  
var C1 = React.createClass({
  render: function() {
    return (
      <C2 url='' />
    )
  }
});

// 在ES6里，通过定义一个继承自React.Component的class来定义一个组件类，像这样：
var React = require('react'); 
class C1 extends React.Component {
  render() {
    return (
      <C2 url={ this.props.url }/>
    )
  }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

## 4. 给组件定义方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// ES5的写法
var React = require('react'); 
var C1 = React.createClass({
  componentWillMount: function() {
    
  },
  render: function() {
    return (
      <C2 url={ this.props.url } />
    )
  }
})
// ES6
var React = require('react'); 
class C1 extends React.Component {
  componentWillMount() {

  }
  render() {
    return (
      <div>222</div>
    )
  }
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

## 5. 定义组件的属性类型 和 默认属性

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 在ES5里，属性类型和默认属性分别通过propTypes成员和getDefaultProps方法来实现
var React = require('react'); 
var A = React.createClass({
  getDefaultProps: function() {
    return {
      a: false,
      b: 10,
    };
  },
  propTypes: {
    a: React.PropTypes.bool.isRequired,
    b: React.PropTypes.number.isRequired,
    c: React.PropTypes.string.isRequired,
    d: React.PropTypes.string.isRequired,
  },
  render: function() {
    return (
      <C />
    );
  }
});
// 在ES6里，可以统一使用static成员来实现
var React = require('react'); 
//ES6
class A extends React.Component {
  static defaultProps = {
    a: false,
    b: 10,
  };  
  static propTypes = {
    a: React.PropTypes.bool.isRequired,
    b: React.PropTypes.number.isRequired,
    c: React.PropTypes.string.isRequired,
    d: React.PropTypes.string.isRequired,
  };  
  // 注意这里有分号
  render() {
    return (
      <C />
    );
  } // 注意这里既没有分号也没有逗号
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

## 6. 初始化STATE

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//ES5 
var React = require('react'); 
var A = React.createClass({
  getInitialState: function() {
    return {
      loopsRemaining: this.props.maxLoops,
    };
  }
})
//ES6
var React = require('react'); 
class A extends React.Component {
  constructor(props){
    super(props);
    this.state = {
      loopsRemaining: this.props.maxLoops,
    };
  }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

## 7. 把方法作为回调提供

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//ES5
var React = require('react'); 
var A = React.createClass({
  handleOptionsButtonClick: function(e) {
    this.setState({showOptionsModal: true});
  },
  render: function(){
    return (
      <TouchableHighlight onClick={this.handleOptionsButtonClick}>
          <Text>{this.props.label}</Text>
      </TouchableHighlight>
    )
  }
});

// 在ES6下，你需要通过bind来绑定this引用，或者使用箭头函数（它会绑定当前scope的this引用）来调用
//ES6

var React = require('react'); 
class PostInfo extends React.Component{
  handleOptionsButtonClick(e){
    this.setState({showOptionsModal: true});
  }
  render(){
    return (
      <TouchableHighlight 
          onPress={this.handleOptionsButtonClick.bind(this)}
          onPress={e=>this.handleOptionsButtonClick(e)}
          >
        <Text>{this.props.label}</Text>
      </TouchableHighlight>
    )
  }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 