## 学习网站

es6网站学习  [阮一峰](http://www.ruanyifeng.com/)

babel网站：https://www.babeljs.cn/

chrome的web前端助手（FeHelper）https://chrome.google.com/webstore/detail/web%E5%89%8D%E7%AB%AF%E5%8A%A9%E6%89%8Bfehelper/pkgccpejnmalmdinmhkkfafefagiiiad?utm_source=inline-install-disabled





## ES6-Babel-Browserify使用教程

1. 定义package.json文件
  ```
  {
    "name" : "es6-babel-browserify",
    "version" : "1.0.0"
  }
  ```
2. 安装babel-cli, babel-preset-es2015和browserify
  * npm install babel-cli browserify -g
  * npm install babel-preset-es2015 --save-dev 
3. 定义.babelrc文件 表示将es6语法转换为es2015
  ```
  {
    "presets": ["es2015"]
  }
  ```
4. 编码
  * js/src/module1.js
    ```
    //分别暴露的方式
    export function foo() {
      console.log('module1 foo()');
    }
    export let bar = function () {
      console.log('module1 bar()');
    }
    export const DATA_ARR = [1, 3, 5, 1]
    ```
  * js/src/module2.js
    ```
    //集中暴露的方式
    let data = 'module2 data'

    function fun1() {
      console.log('module2 fun1() ' + data);
    }

    function fun2() {
      console.log('module2 fun2() ' + data);
    }

    export {fun1, fun2}
    ```
  * js/src/module3.js
    ```
    //默认暴露的方式
    export default {
      name: 'Tom',
      setName: function (name) {
        this.name = name
      }
    }
    ```
  * js/src/app.js
    ```
    import {foo, bar} from './module1'
    import {DATA_ARR} from './module1'
    import {fun1, fun2} from './module2'
    import person from './module3'

    import $ from 'jquery'

    $('body').css('background', 'red')

    foo()
    bar()
    console.log(DATA_ARR);
    fun1()
    fun2()

    person.setName('JACK')
    console.log(person.name);
    ```
5. 编译
  * 使用Babel将ES6编译为ES5代码(但包含CommonJS语法) : **babel js/src -d js/lib**
  * 使用Browserify编译js : **browserify js/lib/app.js -o js/lib/bundle.js**
6. 页面中引入测试
  ```
  <script type="text/javascript" src="js/lib/bundle.js"></script>
  ```
7. 引入第三方模块(jQuery)
  1). 下载jQuery模块: 
    * npm install jquery@1 --save
        2). 在app.js中引入并使用
    ```
    import $ from 'jquery'
    $('body').css('background', 'red')
    ```




## 暴露方式

### 分别暴露

```js
//分别暴露的方式
export function foo() {
  console.log('module1 foo()');
}
export let bar = function () {
  console.log('module1 bar()');
}
export const DATA_ARR = [1, 3, 5, 1]
```

### 集中暴露

```js
//集中暴露的方式
let data = 'module2 data'

function fun1() {
  console.log('module2 fun1() ' + data);
}

function fun2() {
  console.log('module2 fun2() ' + data);
}

export {fun1, fun2}
```

### 默认暴露

```js
//默认暴露的方式
export default {
  name: 'Tom',
  setName: function (name) {
    this.name = name
  }
}
```

**默认暴露，只能暴露一个对象**

但我们可以在对象中，包裹其他对象就达到暴露多个对象的目的了。



### 引入模块

#### 导入非默认暴露

**当不是默认暴露时**，就需要使用 对象的析构 来接收。

```js
import {foo, bar} from './module1'
import {fun1, fun2} from './module2'
```

import {对象名或方法名}  from 路径

对象名和方法名的名字都必须和暴露的名字一样。

#### 导入默认暴露

接收默认暴露时，接收到的对象，就是暴露对象的类型。

```js
import person from './module3'


person.setName('JACK')
console.log(person.name);
```

#### 引入Jquery

npm install jquery@1

@后表示输入版本号，1表示下载为1的最新版本

1.1 表示下载1.1 之后的最新版本

```xml
F:\WebStormSpace\06_ES6_Babel_Browserify>npm install jquery@1

npm WARN es6-babel-browserify@1.0.0 No description
npm WARN es6-babel-browserify@1.0.0 No repository field.
npm WARN es6-babel-browserify@1.0.0 No license field.

+ jquery@1.12.4
added 1 package from 1 contributor in 10.564s

```

这里就下载为1.12.4 版本了。

下载完成后，package.json中就会添加了一个依赖。

```json
{
  "name": "es6-babel-browserify",
  "version": "1.0.0",
  "devDependencies": {
    "babel-preset-es2015": "^6.24.1"
  },
  "dependencies": {
    "jquery": "^1.12.4"  //这里就是刚刚下载的版本
  }
}

```



 











