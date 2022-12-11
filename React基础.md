# React基础

## React组件
### 1. 组建的使用
**1.1 函数式组件**

定义组件最简单的方式就是编写 JavaScript 函数：
```js
 //1.创建函数式组件
      function MyComponent(props) {
        console.log(this) //此处的this是undefined，因为babel编译后开启了严格模式
        return <h2>我是用函数定义的组件(适用于【简单组件】的定义)</h2>
      }

 //2.渲染组件到页面
      ReactDOM.render(<MyComponent />, document.getElementById('test'))
```
该函数是一个有效的 React 组件，因为它接收唯一带有数据的 “props”（代表属性）对象与并返回一个 React 元素。这类组件被称为“函数组件”，因为它本质上就是 JavaScript 函数。

让我们来回顾一下这个例子中发生了什么：
1. React解析组件标签，找到了MyComponent组件。
2. 发现组件是使用函数定义的，随后调用该函数，将返回的虚拟DOM转为真实DOM，随后呈现在页面中。

注意： 组件名称必须以大写字母开头。

React 会将以小写字母开头的组件视为原生 DOM 标签。例如，`<div />` 代表 HTML 的 div 标签，而 `<Welcome /> `则代表一个组件，并且需在作用域内使用 Welcome。

你可以在深入 [JSX](https://zh-hans.reactjs.org/docs/jsx-in-depth.html#user-defined-components-must-be-capitalized) 中了解更多关于此规范的原因

**1.2类组建**

将函数组件转换成 class 组件

通过以下五步将 `Clock` 的函数组件转成 class 组件：

1. 创建一个同名的 ES6 class，并且继承于 React.Component。
2. 添加一个空的 render() 方法。
3. 将函数体移动到 render() 方法之中。
4. 在 render() 方法中使用 this.props 替换 props。
删除剩余的空函数声明。
```js
class MyComponent extends React.Component {
  render() {
    console.log('render中的this:', this)
    return <h2>我是用类定义的组件(适用于【复杂组件】的定义)</h2>
  }
}

ReactDOM.render(<MyComponent />, document.getElementById('test'))
```
每次组件更新时 `render` 方法都会被调用，但只要在相同的 DOM 节点中渲染 `<MyComponent/> ，就仅有一个 `MyComponent` 组件的 class 实例被创建使用。这就使得我们可以使用如 state 或生命周期方法等很多其他特性。

执行过程：

1. React解析组件标签，找到相应的组件
2. 发现组件是类定义的，随后new出来的类的实例，并通过该实例调用到原型上的render方法
3. 将render返回的虚拟DOM转化为真实的DOM,随后呈现在页面中

### 2. state
**2.1 基本使用**
> 我们都说React是一个状态机，体现是什么地方呢，就是体现在state上，通过与用户的交互，实现不同的状态，然后去渲染UI,这样就让用户的数据和界面保持一致了。state是组件的私有属性。  
在React中，更新组件的state，结果就会重新渲染用户界面(不需要操作DOM),一句话就是说，用户的界面会随着状态的改变而改变。  
state是组件对象最重要的属性，值是对象（可以包含多个key-value的组合）  
简单的说就是组件的状态，也就是该组件所存储的数据

案例：

需求：页面显示【今天天气很炎热】，鼠标点击文字的时候，页面更改为【今天天气很凉爽】

核心代码如下：
```js
<body>
    <!-- 准备好容器 -->
    <div id="test">
        
    </div>
</body>

<!--这里使用了js来创建虚拟DOM-->
<script type="text/babel">
        //1.创建组件
        class St extends React.Component{
            constructor(props){
                super(props);
                //先给state赋值
                this.state = {isHot:true,win:"ss"};
                //找到原型的dem，根据dem函数创建了一个dem1的函数，并且将实例对象的this赋值过去
                this.dem1 = this.dem.bind(this);
            }
            //render会调用1+n次【1就是初始化的时候调用的，n就是每一次修改state的时候调用的】
            render(){ //这个This也是实例对象
                //如果加dem()，就是将函数的回调值放入这个地方
                //this.dem这里面加入this，并不是调用，只不过是找到了dem这个函数，在调用的时候相当于直接调用，并不是实例对象的调用
                return <h1 onClick = {this.dem1}>今天天气很{this.state.isHot?"炎热":"凉爽"}</h1>    
            }
            //通过state的实例调用dem的时候，this就是实例对象
            dem(){
                const state =  this.state.isHot;
                 //状态中的属性不能直接进行更改，需要借助API
                // this.state.isHot = !isHot; 错误
                //必须使用setState对其进行修改，并且这是一个合并
                this.setState({isHot:!state});
            }
        }
        // 2.渲染，如果有多个渲染同一个容器，后面的会将前面的覆盖掉
        ReactDOM.render(<St/>,document.getElementById("test"));
</script>
```
在类式组件的函数中，直接修改state值
```js
this.state.isHot = false
```
这时候会发现页面内容不会改变，原因是 React 中不建议 `state`不允许直接修改，而是通过类的原型对象上的方法 `setState()`

注意：

1. 组件的构造函数，必须要传递一个props参数
2. 特别关注`this`【重点】，类中所有的方法局部都开启了严格模式，如果直接进行调用，this就是undefined
3. 想要改变state,需要使用`setState`进行修改，如果只是修改state的部分属性，则不会影响其他的属性，这个只是合并并不是覆盖。

`在优化过程中遇到的问题`

1. 组件中的 render 方法中的 this 为组件实例对象
2. 组件自定义方法中由于开启了严格模式，this 指向undefined如何解决
  1. 通过 bind 改变 this 指向
  2. 推荐采用箭头函数，箭头函数的 this 指向
3. state 数据不能直接修改或者更新

##  组件通信的方式
1. 父子组件通信方式