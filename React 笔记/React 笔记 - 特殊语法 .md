# React 特殊语法

## 1. () 的用法

```js
        //这个箭头函数就是一个action Creator
        let increase = (amount) => ({type:INC,amount});
        //等价于
        //let increase = (amount) => { return {type:INC,amount}};

   一个（）相当于一个return
```

## 2.三点用法

```js
 ... 的作用
 1.打包
 function fn(...param){}  fn(1,2,3) 将1,2,3 打包为数组
 2.解包
 const arr1 = [3,4,5] const arr2 = [1,2,...arr1,6,7] 最终解包为[1,2,3,4,5,6,7]

3. 传递所有的属性

let user = {age:18,name:'yxq'};
let state = user;
let state1 = {...user}; //这句话，相当于把user中的属性，解构成每个属性给state1

console.log(state);  //输出 {age:18,name:'yxq'}
console.log(state1); //输出 {age:18,name:'yxq'}

//下面这个例子一样的原理

export default class MyNavLink extends Component {
    render() {
        return (
            <NavLink  {...this.props}   activeClassName='activeClass' />
        )
    }
}

//{...this.props} 这个接包和压缩包，就可以将MyNavLink中的 所有属性都传递给NavLink 中

```

## 3.多个箭头函数

```js
        //   (a) => {return a;} 这个是一个箭头函数
        // connect =  () => 箭头函数 ，表示第一个匿名函数（无参数）返回的是一个箭头函数，而不是一个对象
        
        let connect = () => (a) => {return a;} 
        let b = connect()('abc');
        console.log(b); //输出 'abc'

//三层箭头函数，第一层的参数可以一直传递下去
let connect = () => (a) => (b) => {return a + b ;}
let b = connect()(3)(7);
console.log(b); //输出 10
```



## 4. || 运算符

```js
import {INC,DEC} from '../actions'

//假如：action对象为{type:'INC'，amount:2)
const reducer = (state = {number: 0}, action) => {
    if (action === undefined) return state; //当初始化时，没有传入action时，取默认值

    switch (action.type) {
        case INC:
            return {number: state.number + (action.amount || 1) }; //这里的 || 表示如果action.amount 为undefined时，取默认值为1
        case DEC:
            return {number: state.number - action.amount};
        default:
            return state;
    }
};

export default reducer;
```


