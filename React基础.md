# React基础

## React组件
### 1. 函数式组件

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

### 2. 类组建

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

-----------------------------------------

## 状态state
### 1. 基本使用
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

### 2. setState()

this.setState()，该方法接收两种参数：对象或函数。
```js
this.setState(partialState, [callback]);
```
- partialState: 需要更新的状态的部分对象
- callback: 更新完状态后的回调函数

有两种写法:

1. 对象：即想要修改的state
```js
this.setState({
    isHot: false
})
```
2. 函数：接收两个函数，第一个函数接受两个参数，第一个是当前state，第二个是当前props，该函数返回一个对象，和直接传递对象参数是一样的，就是要修改的state；第二个函数参数是state改变后触发的回调
```js
this.setState(state => ({count: state.count+1});
```
- 在执行 `setState`操作后，React 会自动调用一次 render()
- render() 的执行次数是 1+n (1 为初始化时的自动调用，n 为状态更新的次数)

### 3. State 的更新可能是异步的

React控制之外的事件中调用setState是同步更新的。比如原生js绑定的事件，setTimeout/setInterval等。

大部分开发中用到的都是React封装的事件，比如onChange、onClick、onTouchMove等，这些事件处理程序中的setState都是异步处理的。
```js
//1.创建组件
class St extends React.Component{
    //可以直接对其进行赋值
    state = {isHot:10};
    render(){ //这个This也是实例对象
        return <h1 onClick = {this.dem}>点击事件</h1> 
    }
//箭头函数 [自定义方法--->要用赋值语句的形式+箭头函数]
    dem = () =>{
        //修改isHot
        this.setState({ isHot: this.state.isHot + 1})
        console.log(this.state.isHot);
    }
}
```
上面的案例中预期setState使得isHot变成了11，输出也应该是11。然而在控制台打印的却是10，也就是并没有对其进行更新。这是因为异步的进行了处理，在输出的时候还没有对其进行处理。
```js
document.getElementById("test").addEventListener("click",()=>{
        this.setState({isHot: this.state.isHot + 1});
        console.log(this.state.isHot);
    })
}
```
但是通过这个原生JS的，可以发现，控制台打印的就是11，也就是已经对其进行了处理。也就是进行了同步的更新。

React怎么调用同步或者异步的呢？

在 React 的 setState 函数实现中，会根据一个变量 isBatchingUpdates 判断是直接更新 this.state 还是放到队列中延时更新，而 isBatchingUpdates 默认是 false，表示 setState 会同步更新 this.state；但是，有一个函数 batchedUpdates，该函数会把 isBatchingUpdates 修改为 true，而当 React 在调用事件处理函数之前就会先调用这个 batchedUpdates将isBatchingUpdates修改为true，这样由 React 控制的事件处理过程 setState 不会同步更新 this.state。

如果是同步更新，每一个setState对调用一个render，并且如果多次调用setState会以最后调用的为准，前面的将会作废；如果是异步更新，多个setSate会统一调用一次render
```js
dem = () =>{
    this.setState({
        isHot:  1,
        cont:444
    })
    this.setState({
    	isHot: this.state.isHot + 1

    })
    this.setState({
        isHot:  888,
        cont:888
    })
}
```

### 4. 异步更新解决方案
出于性能考虑，React 可能会把多个 `setState()` 调用合并成一个调用。

因为 `this.props` 和 `this.state` 可能会异步更新，所以你不要依赖他们的值来更新下一个状态。

例如，此代码可能会无法更新计数器：
```js
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```
要解决这个问题，可以让 `setState()` 接收一个函数而不是一个对象。这个函数用上一个 state 作为第一个参数，将此次更新被应用时的 props 做为第二个参数：
```js
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```
上面使用了箭头函数，不过使用普通的函数也同样可以：
```js
// Correct
this.setState(function(state, props) {
  return {
    counter: state.counter + props.increment
  };
});
```

----------------------------------------

## Props
与state不同，state是组件自身的状态，而props则是外部传入的数据

基本使用：
```js
<body>
    <div id = "div">

    </div>

</body>
<script type="text/babel">
    class Person extends React.Component{
        render(){
          const { name, age, sex } = this.props
            return (
                <ul>
                  <li>姓名：{name}</li>
                  <li>性别：{sex}</li>
                  <li>年龄：{age + 1}</li>
                </ul>
          )
        }
    }
    //传递数据
    ReactDOM.render(<Person name="tom" age = {41} sex="男"/>,document.getElementById("div"));
</script>
```
如果传递的数据是一个对象，可以更加简便的使用
```js
<script type="text/babel">
    class Person extends React.Component{
        render(){
            return (
                <ul>
                    <li>{this.props.name}</li>
                    <li>{this.props.age}</li>
                    <li>{this.props.sex}</li>
                </ul>
            )
        }
    }
    const p = {name:"张三",age:"18",sex:"女"}
   ReactDOM.render(<Person {...p}/>,document.getElementById("div"));
</script>
```
### 2. props限制
> 注意：
自 React v15.5 起，`React.PropTypes` 已移入另一个包中。请使用 [prop-types](https://www.npmjs.com/package/prop-types) 库 代替。
我们提供了一个 [codemod](https://zh-hans.reactjs.org/blog/2017/04/07/react-v15.5.0.html#migrating-from-reactproptypes) 脚本来做自动转换。

随着你的应用程序不断增长，你可以通过类型检查捕获大量错误。对于某些应用程序来说，你可以使用 `Flow` 或 `TypeScript` 等 JavaScript 扩展来对整个应用程序做类型检查。但即使你不使用这些扩展，React 也内置了一些类型检查的功能。要在组件的 props 上进行类型检查，你只需配置特定的`propTypes` 属性：

react对此提供了相应的解决方法：

- propTypes:类型检查，还可以限制不能为空
- defaultProps：默认值
  
> 从 ES2022 开始，你也可以在 React 类组件中将 `defaultProps` 声明为静态属性。欲了解更多信息，请参阅 [class public static fields](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Public_class_fields#public_static_fields)。这种现代语法需要添加额外的编译步骤才能在老版浏览器中工作。
```js

<!-- 准备好一个“容器” -->
<div id="test1"></div>
<div id="test2"></div>
<div id="test3"></div>

<script type="text/babel">
    //创建组件
    class Person extends React.Component{
        render(){
            // console.log(this);
            const {name,age,sex} = this.props
            //props是只读的
            //this.props.name = 'jack' //此行代码会报错，因为props是只读的
            return (
                <ul>
                    <li>姓名：{name}</li>
                    <li>性别：{sex}</li>
                    <li>年龄：{age+1}</li>
                </ul>
            )
        }
    }
    //对标签属性进行类型、必要性的限制
    Person.propTypes = {
        name:PropTypes.string.isRequired, //限制name必传，且为字符串
        sex:PropTypes.string,//限制sex为字符串
        age:PropTypes.number,//限制age为数值
        speak:PropTypes.func,//限制speak为函数
    }
    //指定默认标签属性值
    Person.defaultProps = {
        sex:'男',//sex默认值为男
        age:18 //age默认值为18
    }
    //渲染组件到页面
    ReactDOM.render(<Person name={100} speak={speak}/>,document.getElementById('test1'))
    ReactDOM.render(<Person name="tom" age={18} sex="女"/>,document.getElementById('test2'))

    const p = {name:'老刘',age:18,sex:'女'}
    // console.log('@',...p);
    // ReactDOM.render(<Person name={p.name} age={p.age} sex={p.sex}/>,document.getElementById('test3'))
    ReactDOM.render(<Person {...p}/>,document.getElementById('test3'))

    function speak(){
        console.log('我说话了');
    }
</script>
```
当传入的 `prop` 值类型不正确时，JavaScript 控制台将会显示警告。出于性能方面的考虑，`propTypes` 仅在开发模式下进行检查。

`defaultProps` 用于确保 this.props.sex 在父组件没有指定其值时，有一个默认值。propTypes 类型检查发生在 defaultProps 赋值后，所以类型检查也适用于 defaultProps。

PropTypes

以下提供了使用不同验证器的例子：
```js
import PropTypes from 'prop-types';

MyComponent.propTypes = {
  // 你可以将属性声明为 JS 原生类型，默认情况下
  // 这些属性都是可选的。
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,

  // 任何可被渲染的元素（包括数字、字符串、元素或数组）
  // (或 Fragment) 也包含这些类型。
  optionalNode: PropTypes.node,

  // 一个 React 元素。
  optionalElement: PropTypes.element,

  // 一个 React 元素类型（即，MyComponent）。
  optionalElementType: PropTypes.elementType,

  // 你也可以声明 prop 为类的实例，这里使用
  // JS 的 instanceof 操作符。
  optionalMessage: PropTypes.instanceOf(Message),

  // 你可以让你的 prop 只能是特定的值，指定它为
  // 枚举类型。
  optionalEnum: PropTypes.oneOf(['News', 'Photos']),

  // 一个对象可以是几种类型中的任意一个类型
  optionalUnion: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message)
  ]),

  // 可以指定一个数组由某一类型的元素组成
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),

  // 可以指定一个对象由某一类型的值组成
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),

  // 可以指定一个对象由特定的类型值组成
  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number
  }),

  // 具有额外属性警告的对象
  optionalObjectWithStrictShape: PropTypes.exact({
    name: PropTypes.string,
    quantity: PropTypes.number
  }),

  // 你可以在任何 PropTypes 属性后面加上 `isRequired` ，确保
  // 这个 prop 没有被提供时，会打印警告信息。
  requiredFunc: PropTypes.func.isRequired,

  // 任意类型的必需数据
  requiredAny: PropTypes.any.isRequired,

  // 你可以指定一个自定义验证器。它在验证失败时应返回一个 Error 对象。
  // 请不要使用 `console.warn` 或抛出异常，因为这在 `oneOfType` 中不会起作用。
  customProp: function(props, propName, componentName) {
    if (!/matchme/.test(props[propName])) {
      return new Error(
        'Invalid prop `' + propName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  },

  // 你也可以提供一个自定义的 `arrayOf` 或 `objectOf` 验证器。
  // 它应该在验证失败时返回一个 Error 对象。
  // 验证器将验证数组或对象中的每个值。验证器的前两个参数
  // 第一个是数组或对象本身
  // 第二个是他们当前的键。
  customArrayProp: PropTypes.arrayOf(function(propValue, key, componentName, location, propFullName) {
    if (!/matchme/.test(propValue[key])) {
      return new Error(
        'Invalid prop `' + propFullName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  })
};
```
限制单个元素

你可以通过 PropTypes.element 来确保传递给组件的 children 中只包含一个元素。
```js
import PropTypes from 'prop-types';

class MyComponent extends React.Component {
  render() {
    // 这必须只有一个元素，否则控制台会打印警告。
    const children = this.props.children;
    return (
      <div>
        {children}
      </div>
    );
  }
}

MyComponent.propTypes = {
  children: PropTypes.element.isRequired
};
```

### 3. 简写方式
```js
<!-- 准备好一个“容器” -->
<div id="test1"></div>
<div id="test2"></div>
<div id="test3"></div>


<script type="text/babel">
    //创建组件
    class Person extends React.Component{

        constructor(props){
            //构造器是否接收props，是否传递给super，取决于：是否希望在构造器中通过this访问props
            // console.log(props);
            super(props)
            console.log('constructor',this.props);
        }

        //对标签属性进行类型、必要性的限制
        static propTypes = {
            name:PropTypes.string.isRequired, //限制name必传，且为字符串
            sex:PropTypes.string,//限制sex为字符串
            age:PropTypes.number,//限制age为数值
        }

        //指定默认标签属性值
        static defaultProps = {
            sex:'男',//sex默认值为男
            age:18 //age默认值为18
        }

        render(){
            // console.log(this);
            const {name,age,sex} = this.props
            //props是只读的
            //this.props.name = 'jack' //此行代码会报错，因为props是只读的
            return (
                <ul>
                    <li>姓名：{name}</li>
                    <li>性别：{sex}</li>
                    <li>年龄：{age+1}</li>
                </ul>
            )
        }
    }

    //渲染组件到页面
    ReactDOM.render(<Person name="jerry"/>,document.getElementById('test1'))
</script>
```
在使用的时候可以通过 `this.props`来获取值 类式组件的 props:

1. 通过在组件标签上传递值，在组件中就可以获取到所传递的值
2. 在构造器里的`props`参数里可以获取到 `props`
3. 可以分别设置 `propTypes` 和 `defaultProps` 两个属性来分别操作 `props`的规范和默认值，两者都是直接添加在类式组件的原型对象上的（所以需要添加 `static`）
4. 同时可以通过...运算符来简化

### 4. props 的只读性
组件无论是使用`函数声明`还是通过 `class 声明`，都绝不能修改自身的 props。来看下这个 sum 函数：
```js
function sum(a, b) {
  return a + b;
}
```
这样的函数被称为[“纯函数”](https://en.wikipedia.org/wiki/Pure_function)，因为该函数不会尝试更改入参，且多次调用下相同的入参始终返回相同的结果。

相反，下面这个函数则不是纯函数，因为它更改了自己的入参：
```js
function withdraw(account, amount) {
  account.total -= amount;
}
```
React 非常灵活，但它也有一个严格的规则：

所有 React 组件都必须像纯函数一样保护它们的 props 不被更改。

当然，应用程序的 UI 是动态的，并会伴随着时间的推移而变化。state在不违反上述规则的情况下，state 允许 React 组件随用户操作、网络响应或者其他变化而动态更改输出内容。

---------------------------------

## refs
`Refs` 提供了一种方式，允许我们访问 DOM 节点或在 `render` 方法中创建的 React 元素。

在典型的 React 数据流中，`props` 是父组件与子组件交互的唯一方式。要修改一个子组件，你需要使用新的 props 来重新渲染它。但是，在某些情况下，你需要在典型数据流之外强制修改子组件。被修改的子组件可能是一个 React 组件的实例，也可能是一个 DOM 元素。对于这两种情况，React 都提供了解决办法。
> 在我们正常的操作节点时，需要采用DOM API 来查找元素，但是这样违背了 React 的理念，因此有了refs

何时使用 Refs

下面是几个适合使用 refs 的情况：

- 管理焦点，文本选择或媒体播放。
- 触发强制动画。
- 集成第三方 DOM 库。
避免使用 refs 来做任何可以通过声明式实现来完成的事情。

有三种操作`refs`的方法，分别为：
- 字符串形式
- 回调形式
- `createRef`形式

### 1. 字符串形式
在想要获取到一个DOM节点，可以直接在这个节点上添加ref属性。利用该属性进行获取该节点的值。

案例：给需要的节点添加ref属性，此时该实例对象的refs上就会有这个值。就可以利用实例对象的refs获取已经添加节点的值
```js
<input ref="dian" type="text" placeholder="点击弹出" />

inputBlur = () =>{
  alert(this.refs.dian.value);
}
```
注意

不建议使用它，因为 string 类型的 refs 存在 [一些问题](https://github.com/facebook/react/pull/8333#issuecomment-271648615)。它已过时并可能会在未来的版本被移除。

如果你目前还在使用 `this.refs.textInput` 这种方式访问 refs ，我们建议用[回调函数](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html#callback-refs) 或 [createRef API](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html#creating-refs) 的方式代替。

### 2. 回调形式
React 也支持另一种设置 refs 的方式，称为“回调 refs”。它能助你更精细地控制何时 refs 被设置和解除。

这种方式会将该DOM作为参数传递过去。

组件实例的`ref`属性传递一个回调函数`c => this.input1 = c` （箭头函数简写），这样会在实例的属性中存储对DOM节点的引用，使用时可通过`this.input1`来使用
```js
<input ref={e => this.input1 = e } type="text" placeholder="点击按钮提示数据"/>
```
`e`会接收到当前节点作为参数，然后将当前节点赋值给实例的`input1`属性上面

下面的例子描述了一个通用的范例：使用 ref 回调函数，在实例的属性中存储对 DOM 节点的引用。
```js
//创建组件
class Demo extends React.Component {
    state = { isHot: false }
	// 在实例上面创建一个函数
    setTextInputRef = e => {
      this.input1 = e
    }

    changeWeather = () => {
      console.log(this.input1)
      //获取原来的状态
      const { isHot } = this.state
      //更新状态
      this.setState({ isHot: !isHot })
    }

    render() {
      const { isHot } = this.state
      return (
        <div>
          <h2>今天天气很{isHot ? '炎热' : '凉爽'}</h2>
          <input ref={this.setTextInputRef} type="text" />
          <br />
          <button onClick={this.changeWeather}>点我切换天气</button>
        </div>
      )
    }
}
```
React 将在组件挂载时，会调用 `ref` 回调函数并传入 DOM 元素，当卸载时调用它并传入 null。

你可以在组件间传递回调形式的 `refs`，就像你可以传递通过 `React.createRef()` 创建的对象 refs 一样。
```js
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />    
     </div>
  );
}

class Parent extends React.Component {
  render() {
    return (
      <CustomTextInput
        inputRef={el => this.inputElement = el}      />
    );
  }
}
```
在上面的例子中，`Parent` 把它的 refs 回调函数当作 `inputRef` props 传递给了 `CustomTextInput`，而且 CustomTextInput 把相同的函数作为特殊的 ref 属性传递给了 `<input>`。结果是，在 Parent 中的 `this.inputElement` 会被设置为与 CustomTextInput 中的 input 元素相对应的 DOM 节点。

### 3 createRef 形式（推荐写法）
创建 Refs

Refs 是使用 `React.createRef()` 创建的，并通过 ref 属性附加到 React 元素。在构造组件时，通常将 Refs 分配给实例属性，以便可以在整个组件中引用它们。
```js
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  render() {
    return <div ref={this.myRef} />;
  }
}
```
访问 Refs

当 ref 被传递给 `render` 中的元素时，对该节点的引用可以在 ref 的 current 属性中被访问。
```js
const node = this.myRef.current;
```
ref 的值根据节点的类型而有所不同：

- 当 ref 属性用于 HTML 元素时，构造函数中使用 `React.createRef()` 创建的 ref 接收底层 DOM 元素作为其 `current` 属性。
- 当 ref 属性用于自定义 class 组件时，`ref` 对象接收组件的挂载实例作为其 `current` 属性。
- 你不能在函数组件上使用 ref 属性，因为他们没有实例。

### 4. 为 DOM 元素添加 ref
以下代码使用 ref 去存储 DOM 节点的引用：
```js
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    // 创建一个 ref 来存储 textInput 的 DOM 元素
    this.textInput = React.createRef();    
    this.focusTextInput = this.focusTextInput.bind(this);
  }

  focusTextInput() {
    // 直接使用原生 API 使 text 输入框获得焦点
    // 注意：我们通过 "current" 来访问 DOM 节点
    this.textInput.current.focus();  
  }

  render() {
    // 告诉 React 我们想把 <input> ref 关联到
    // 构造器里创建的 `textInput` 上
    return (
      <div>
        <input
          type="text"
          ref={this.textInput} 
        />        
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```
React 会在组件挂载时给 `current` 属性传入 DOM 元素，并在组件卸载时传入 `null` 值。`ref` 会在 `componentDidMount` 或 `componentDidUpdate` 生命周期钩子触发前更新。

注意：我们不要过度的使用 ref，如果发生时间的元素刚好是需要操作的元素，就可以使用事件对象去替代。

### 5. 为 class 组件添加 Ref
如果我们想包装上面的 `CustomTextInput`，来模拟它挂载之后立即被点击的操作，我们可以使用 ref 来获取这个自定义的 input 组件并手动调用它的 `focusTextInput` 方法：
```js
class AutoFocusTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = React.createRef();  
  }

  componentDidMount() {
    this.textInput.current.focusTextInput();  
  }

  render() {
    return (
      <CustomTextInput ref={this.textInput} />    
    );
  }
}
```
请注意，这仅在 CustomTextInput 声明为 class 时才有效：
```js
class CustomTextInput extends React.Component {  // ...
}
```

### 6. Refs 与函数组件
默认情况下，你不能在函数组件上使用 `ref` 属性，因为它们没有实例：
```js
function MyFunctionComponent() {
  return <input />;
}

class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = React.createRef();
  }
  render() {
    // This will *not* work!
    return (
      <MyFunctionComponent ref={this.textInput} />
    );
  }
}
```
如果要在函数组件中使用 `ref`，你可以使用 [forwardRef](https://zh-hans.reactjs.org/docs/forwarding-refs.html)（可与 [useImperativeHandle](useimperativehandle) 结合使用），或者可以将该组件转化为 class 组件。

不管怎样，你可以在函数组件内部使用 `ref` 属性，只要它指向一个 DOM 元素或 class 组件：
```js
function CustomTextInput(props) {
  // 这里必须声明 textInput，这样 ref 才可以引用它
  const textInput = useRef(null);

  function handleClick() {
    textInput.current.focus();
  }

  return (
    <div>
      <input
        type="text"
        ref={textInput} />
      <input
        type="button"
        value="Focus the text input"
        onClick={handleClick}
      />
    </div>
  );
}
```

---------------------------------

## 事件处理
> React的事件是通过onXxx属性指定事件处理函数
React 使用的是自定义事件，而不是原生的 DOM 事件
React 的事件是通过事件委托方式处理的（为了更加的高效）
可以通过事件的 `event.target`获取发生的 DOM 元素对象，可以尽量减少 `refs`的使用  
事件中必须返回的是函数

### 1. React事件
React 元素的事件处理和 DOM 元素的很相似，但是有一点语法上的不同：
- React 事件的命名采用小驼峰式（camelCase），而不是纯小写。
- 使用 JSX 语法时你需要传入一个函数作为事件处理函数，而不是一个字符串。
例如，传统的 HTML：
```html
<button onclick="activateLasers()">
  Activate Lasers
</button>
```
在 React 中略微不同：
```html
<button onClick={activateLasers}>  
   Activate Lasers
</button>
```
在 React 中另一个不同点是你不能通过返回 false 的方式阻止默认行为。你必须显式地使用 `preventDefault`。例如，传统的 HTML 中阻止表单的默认提交行为，你可以这样写：
```html
<form onsubmit="console.log('You clicked submit.'); return false">
  <button type="submit">Submit</button>
</form>
```
在 React 中，可能是这样的：
```js
function Form() {
  function handleSubmit(e) {
    e.preventDefault();    
    console.log('You clicked submit.');
  }

  return (
    <form onSubmit={handleSubmit}>
      <button type="submit">Submit</button>
    </form>
  );
}
```
在这里，`e` 是一个合成事件。React 根据 [W3C](https://www.w3.org/TR/DOM-Level-3-Events/) 规范来定义这些合成事件，所以你不需要担心跨浏览器的兼容性问题。React 事件与原生事件不完全相同。如果想了解更多，请查看 [SyntheticEvent](https://zh-hans.reactjs.org/docs/events.html) 参考指南。  
使用 React 时，你一般不需要使用 `addEventListener` 为已创建的 DOM 元素添加监听器。事实上，你只需要在该元素初始渲染的时候添加监听器即可。

### 2. 件绑定事件
当你使用 ES6 class 语法定义一个组件的时候，通常的做法是将事件处理函数声明为 class 中的方法。例如，下面的 Toggle 组件会渲染一个让用户切换开关状态的按钮：
```js
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};
    // 为了在回调中使用 `this`，这个绑定是必不可少的    	this.handleClick = this.handleClick.bind(this);  }

  handleClick() {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }
      
  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}
```
你必须谨慎对待 JSX 回调函数中的 `this`，在 JavaScript 中，class 的方法默认不会绑定 this。如果你忘记绑定 `this.handleClick` 并把它传入了 `onClick`，当你调用这个函数的时候 this 的值为 undefined。

这并不是 React 特有的行为；这其实与 JavaScript 函数工作原理有关。通常情况下，如果你没有在方法后面添加 ()，例如 onClick={this.handleClick}，你应该为这个方法绑定 this。

如果觉得使用 `bind` 很麻烦，这里有两种方式可以解决。你可以使用 [public class fields](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Public_class_fields#public_instance_fields) 语法 to correctly bind callbacks:
```js
class LoggingButton extends React.Component {
  // This syntax ensures `this` is bound within handleClick.
  handleClick = () => {
    console.log('this is:', this);
  };
  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
```
[Create React App](https://github.com/facebook/create-react-app) 默认启用此语法。

如果你没有使用 class fields 语法，你可以在回调中使用箭头函数：
```js
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this);
  }

  render() {
    // 此语法确保 `handleClick` 内的 `this` 已被绑定。
    return (
      <button onClick={() => this.handleClick()}>
        Click me
      </button>
    );
  }
}
```
此语法问题在于每次渲染 LoggingButton 时都会创建不同的回调函数。在大多数情况下，这没什么问题，但如果该回调函数作为 prop 传入子组件时，这些组件可能会进行额外的重新渲染。我们通常建议在构造器中绑定或使用 `class fields` 语法来避免这类性能问题。

- 直接在render里写行内的箭头函数(不推荐)
- 在组件内使用箭头函数定义一个方法(推荐)
- 直接在组件内定义一个非箭头函数的方法，然后在render里直接使用 onClick= {this.handleClick.bind(this)} (不推荐)
- 直接在组件内定义一个非箭头函数的方法，然后在constructor里bind(this)(推荐)

### 3. 向事件处理程序传递参数
在循环中，通常我们会为事件处理函数传递额外的参数。例如，若 `id` 是你要删除那一行的 ID，以下两种方式都可以向事件处理函数传递参数：
```html
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```
上述两种方式是等价的，分别通过箭头函数和 `Function.prototype.bind` 来实现。

在这两种情况下，React 的事件对象 `e` 会被作为第二个参数传递。如果通过箭头函数的方式，事件对象必须显式的进行传递，而通过 `bind` 的方式，事件对象以及更多的参数将会被隐式的进行传递。


---------------------------------

## 受控和非受控组件
**受控组件：**

使 React 的 state 成为“唯一数据源”。渲染表单的 React 组件还控制着用户输入过程中表单发生的操作。被 React 以这种方式控制取值的表单输入元素就叫做“受控组件”
```js
saveName = (event) =>{
    this.setState({name:event.target.value});
}

savePwd = (event) => {
    this.setState({pwd:event.target.value});
}

render() {
    return (
        <form action="http://www.baidu.com" onSubmit={this.login}>
            用户名：<input value={this.state.name} onChange={this.saveName} type = "text" />
            密码<input value={this.state.pwd} onChange={this.savePwd} type = "password"/>
            <button>登录</button>
        </form>
    )
}
```
由于在表单元素上设置了 `value` 属性，因此显示的值将始终为 `this.state.value`，这使得 React 的 state 成为唯一数据源。由于 `onchange` 在每次按键时都会执行并更新 React 的 state，因此显示的值将随着用户输入而更新。

对于受控组件来说，输入的值始终由 React 的 state 驱动。

**非受控组件：**

非受控组件其实就是表单元素的值不会更新state。输入数据都是现用现取的。

如下：下面并没有使用state来控制属性，使用的是事件来控制表单的属性值。
> 表单提交同样需要通过事件来处理，提交表单的事件通过form标签的onSubmit事件来绑定，处理表单的方式因情况而已，但是一定要注意，必须要取消默认行为，否则会触发表单的默认提交行为：
```js
class Login extends React.Component{

    login = (event) =>{
        event.preventDefault(); //阻止表单默认事件
            console.log(this.name.value);
            console.log(this.pwd.value);
        }
        render() {
            return (
                <form action="http://www.baidu.com" onSubmit={this.login}>
                用户名：<input ref = {e => this.name =e } type = "text" name ="username"/>
                密码：  <input ref = {e => this.pwd =e } type = "password" name ="password"/>
                <button>登录</button>
                </form>
            )
    }
}
```
---------------------------------

##  组件通信的方式
### 1. 父子组件通信方式
- 父传子

  在 React 中，父组件把数据传递给子组件，仍然是通过 props 的方式传递；
```js
import React, { Component } from 'react'
import ReactDOM from 'react-dom'

class Header extends Component {
  render () {
    return (<h1>
      <p>{this.props.data}</p>
    </h1>)
  }
}

// 此时的 Panel 是父组件而 Header 是子组件，父子组件通信时父传子，仍然是通过 props 传递的
class Panel extends Component {
  render () {
    return (<div className="container">
      <p>{this.props.news}</p>
      <Header data={this.props.min} />
    </div>)
  }
}

let data = {
  news: '快下课了',
  min: '拖几分钟'
}

ReactDOM.render(<Panel {...data} />, document.getElementById('root'))
```
- 子传父

在 React 中子组件修改父组件的方式和 Vue 不同；子组件如果想修改父组件的数据，父组件在使用子组件的时候，通过 props 传给子组件一个可以修改父组件的方法，当子组件需要修改父组件的数据时，通过 `this.props` 找到这个方法执行对应的方法
```js
import React, { Component } from 'react'
import ReactDOM from 'react-dom'

import 'bootstrap/dist/css/bootstrap.css'

class Panel extends Component {
  static defaultProps = {
    a: 1
  }
  constructor () {
    super()

    this.state = {
      color: 'success'
    }
  }

  changeColor = (color) => {
    this.setState({
      color
    })
  }

  render () {
    return (<div className="container">
      <div className={`panel panel-${this.state.color}`}>
        <div className="panel-heading">
          {this.props.head}        </div>
        <div className="panel-body">
          {this.props.body}        </div>
        {/*通过 modifyColor 这个 props 把 Panel 组件的 changeColor 方法传递给 Footer 组件*/}        <Footer type={this.state.color}
                modifyColor={this.changeColor} />
      </div>
    </div>)
  }
}

class Footer extends Component {
  change = () => {
    this.props.modifyColor('danger')
  }
  render () {

    return (<div className="panel-footer">
      <button className={`btn btn-${this.props.type}`} onClick={this.change}>变色</button>
    </div>)
  }
}

ReactDOM.render(<Panel head="头信息" body="信息主体"/>, document.getElementById('root'))

// React 同样是单向数据流，即数据只能通过只能从父组件流向子组件
// 所以子组件如果想修改父组件的数据，父组件在使用子组件的时候，通过props传给子组件一个可以修改父组件的方法，当子组件需要修改父组件的数据时，通过this.props 找到这个方法执行对应的方法就可以了
```
-  ref标记 (父组件拿到子组件的引用，从而调用子组件的方法)
  
     1. React.creactRef (引用保存在ref.current属性中)
 ```js
 class Children extends Component {
  public state = {
    name: 'Children',
  };

  public print() {
    console.log('print');
  }

  public render() {
    return <div>{this.state.name}</div>;
  }
}

class Father extends Component {
  public ref = React.createRef<any>();

  public handleClick() {
    this.ref.current.print();
  }

  public render() {
    return (
      <>
        <button onClick={() => this.handleClick()}>click</button>
        <Children ref={this.ref} />
      </>
    );
  }
}
```  
    2. 回调Refs  
    此处有两个示例，其中textInput是对element元素的引用， inputRef是对Input组件的引用
```js
class Input extends Component {
  textInput: HTMLElement | null = null;

  componentDidMount() {
    this.textInput?.focus();
  }

  public showInfo = () => {
    console.log('this is callback ref');
  };

  render() {
    return (
      <div>
        <input
          ref={(element) => {
            this.textInput = element;
          }}
        />
        <button onClick={() => this.handleClick()}>click</button>
      </div>
    );
  }

  private handleClick = () => {
    console.log(this.textInput);
  };
}

class TopInput extends Component {
  public inputRef: Input | null = null;

  public executeChildren = () => {
    this.inputRef?.showInfo();
  };
  render() {
    return (
      <div>
        <Input
          ref={(element) => {
            this.inputRef = element;
          }}
        />
        <button onClick={() => this.executeChildren()}>click_2</button>
      </div>
    );
  }
}
```

### 2.  非父子组件通信方式
- 状态提升(中间人模式)  
React中的状态提升概括来说,就是将多个组件需要共享的状态提升到它们最近的父组件
上.在父组件上改变这个状态然后通过props分发给子组件
```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';


//父组件
class Parent extends React.Component {
    // 提供共享状态
    state = {
        count: 0
    }
    //提供修改状态的方法
    onIncrement = () => {
        this.setState({
            count: this.state.count + 1
        })
    }

   //类组件必须提供render()方法
    render() {
        return (
            <div>
                <Child1 count={this.state.count}/>
                <hr/>
                <Child2 onIncrement={this.onIncrement}/>
            </div>
        )
    }
}

//子组件
const Child1 = (props) => {

    return (
        <div>
            <h2>我是子组件1</h2>
            <h3>计数器：{props.count}</h3>
        </div>
    )
}
const Child2 = (props) => {
    return (
        <div>
            <h2>我是子组件2</h2>
            <button onClick={() => props.onIncrement()}>+1</button>
        </div>
    )
}
ReactDOM.render(<Parent/>, document.getElementById('root'))
```
- 发布订阅模式  
  参考 `eventBus`、 `Redux`
- context状态树传参  
  a. 先定义全局context对象 
  ```js
  import React from 'react' 
  const GlobalContext = React.createContext() 
  export default GlobalContext
  ```
  b. 根组件引入GlobalContext，并使用GlobalContext.Provider（生产者） 
  ```js
  //重新包装根组件 class App {} 
  <GlobalContext.Provider 
    value={{ 
      name:"kerwin", 
      age:100, 
      content:this.state.content, 
      show:this.show.bind(this), 
      hide:this.hide.bind(this) 
    }}
  >
    <之前的根组件></之前的根组件> 
  </GlobalContext.Provider>
  ```
  c. 任意组件引入GlobalContext并调用context，使用GlobalContext.Consumer（消费者） 
  ```js
  <GlobalContext.Consumer> 
    { 
      context => { 
        this.myshow = context.show; //可以在当前组件任意函数触发 
        this.myhide = context.hide;//可以在当前组件任意函数触发 
        return ( 
          <div>
            {context.name}-{context.age}-{context.content} 
          </div> 
        ) 
      }
    } 
  </GlobalContext.Consumer>
  ```
  注意：GlobalContext.Consumer内必须是回调函数，通过context方法改变根组件状态
  > context优缺点：  
  优点：跨组件访问数据  
  缺点：react组件树种某个上级组件shouldComponetUpdate 返回false,当context更新时，不
  会引起下级组件更新

  ----------------------------------

##  React生命周期
1. 初始化阶段
  - componentWillMount
  - render
  - componentDidMout

2. 运行中阶段
   - componentWillReceiveProps
   - shouldComponentUpdate
   - componentWillUpdate
   - render
   - componentDidUpdate

3. 销毁阶段
   - componentWillUnmount


----------------------------------

## React Hooks
使用hooks理由
1. 高阶组件为了复用，导致代码层级复杂
2. 生命周期的复杂
3. 写成functional组件,无状态组件 ，因为需要状态，又改成了class,成本高

### UseState
useState 可以使函数组件像类组件一样拥有 state，函数组件通过 useState 可以让组件重新渲染，更新视图。
```js
const [ ①state , ②dispatch ] = useState(③initData)
```
**useState 基础介绍：**  
① state，目的提供给 UI ，作为渲染视图的数据源。  
② dispatchAction 改变 state   
③ initData 有两种情况，第一种情况是非函数，将作为 state 初始化的值。 第二种情况是函数，函数的返回值作为 useState 初始化的值。

**useState 基础用法：**
```js
const DemoState = (props) => {
   /* number为此时state读取值 ，setNumber为派发更新的函数 */
   let [number, setNumber] = useState(0) /* 0为初始值 */
   return (<div>
       <span>{ number }</span>
       <button onClick={ ()=> {
         setNumber(number+1)
         console.log(number) /* 这里的number是不能够即使改变的  */
       } } ></button>
   </div>)
}
```
**useState 注意事项：**  
① 在函数组件一次执行上下文中，state 的值是固定不变的。
```js
function Index(){
    const [ number, setNumber ] = React.useState(0)
    const handleClick = () => setInterval(()=>{
        // 此时 number 一直都是 0
        setNumber(number + 1 ) 
    },1000)
    return <button onClick={ handleClick } > 点击 { number }</button>
}
```
② 如果两次 dispatchAction 传入相同的 state 值，那么组件就不会更新。
```js
export default function Index(){
    const [ state  , dispatchState ] = useState({ name:'alien' })
    const  handleClick = ()=>{ // 点击按钮，视图没有更新。
        state.name = 'Alien'
        dispatchState(state) // 直接改变 `state`，在内存中指向的地址相同。
    }
    return <div>
         <span> { state.name }</span>
        <button onClick={ handleClick }  >changeName++</button>
    </div>
}
```
③ 当触发 dispatchAction 在当前执行上下文中获取不到最新的 state, 只有在下一次组件 rerender 中才能获取到。

### useReducer
useReducer 是 react-hooks 提供的能够在无状态组件中运行的类似redux的功能 api 。
```js
const [ ①state , ②dispatch ] = useReducer(③reducer)
```
**useState 基础介绍：** 

① 更新之后的 state 值。  
② 派发更新的 dispatchAction 函数, 本质上和 useState 的 dispatchAction 是一样的。  
③ 一个函数 reducer ，我们可以认为它就是一个 redux 中的 reducer , reducer的参数就是常规reducer里面的state和action, 返回改变后的state, 这里有一个需要注意的点就是：**如果返回的 state 和之前的 state ，内存指向相同，那么组件将不会更新。**

**useReducer 基础用法：**
```js
const DemoUseReducer = ()=>{
    /* number为更新后的state值,  dispatchNumbner 为当前的派发函数 */
   const [ number , dispatchNumbner ] = useReducer((state,action)=>{
       const { payload , name  } = action
       /* return的值为新的state */
       switch(name){
           case 'add':
               return state + 1
           case 'sub':
               return state - 1 
           case 'reset':
             return payload       
       }
       return state
   },0)
   return <div>
      当前值：{ number }
      { /* 派发更新 */ }
      <button onClick={()=>dispatchNumbner({ name:'add' })} >增加</button>
      <button onClick={()=>dispatchNumbner({ name:'sub' })} >减少</button>
      <button onClick={()=>dispatchNumbner({ name:'reset' ,payload:666 })} >赋值</button>
      { /* 把dispatch 和 state 传递给子组件  */ }
      <MyChildren  dispatch={ dispatchNumbner } State={{ number }} />
   </div>
}
```

### useEffect
React hooks也提供了 api ，用于弥补函数组件没有生命周期的缺陷。其本质主要是运用了 hooks 里面的 useEffect ， useLayoutEffect，还有 useInsertionEffect。其中最常用的就是 useEffect 。我们首先来看一下 useEffect 的使用。
```js
useEffect 基础介绍：
useEffect(()=>{
    return destory
},dep)
```
- useEffect 第一个参数 callback, 返回的 destory ， destory 作为下一次callback执行之前调用，用于清除上一次 callback 产生的副作用。
- 第二个参数作为依赖项，是一个数组，可以有多个依赖项，依赖项改变，执行上一次callback 返回的 destory ，和执行新的 effect 第一个参数 callback 

对于 useEffect 执行， React 处理逻辑是采用异步调用 ，对于每一个 effect 的 callback， React 会向 setTimeout回调函数一样，放入任务队列，等到主线程任务完成，DOM 更新，js 执行完成，视图绘制完毕，才执行。所以 effect 回调函数不会阻塞浏览器绘制视图。

**useEffect 基础用法：**
```js
/* 模拟数据交互 */
function getUserInfo(a){
    return new Promise((resolve)=>{
        setTimeout(()=>{ 
           resolve({
               name:a,
               age:16,
           }) 
        },500)
    })
}

const Demo = ({ a }) => {
    const [ userMessage , setUserMessage ] :any= useState({})
    const div= useRef()
    const [number, setNumber] = useState(0)
    /* 模拟事件监听处理函数 */
    const handleResize =()=>{}
    /* useEffect使用 ，这里如果不加限制 ，会使函数重复执行，陷入死循环*/
    useEffect(()=>{
       /* 请求数据 */
       getUserInfo(a).then(res=>{
           setUserMessage(res)
       })
       /* 定时器 延时器等 */
       const timer = setInterval(()=>console.log(666),1000)
       /* 操作dom  */
       console.log(div.current) /* div */
       /* 事件监听等 */
       window.addEventListener('resize', handleResize)
         /* 此函数用于清除副作用 */
       return function(){
           clearInterval(timer) 
           window.removeEventListener('resize', handleResize)
       }
    /* 只有当props->a和state->number改变的时候 ,useEffect副作用函数重新执行 ，如果此时数组为空[]，证明函数只有在初始化的时候执行一次相当于componentDidMount */
    },[ a ,number ])
    return (<div ref={div} >
        <span>{ userMessage.name }</span>
        <span>{ userMessage.age }</span>
        <div onClick={ ()=> setNumber(1) } >{ number }</div>
    </div>)
}
```
如上在 useEffect 中做的功能如下：

① 请求数据。  
② 设置定时器,延时器等。  
③ 操作 dom , 在 React Native 中可以通过 ref 获取元素位置信息等内容。  
④ 注册事件监听器, 事件绑定，在 React Native 中可以注册 NativeEventEmitter 。  
⑤ 还可以清除定时器，延时器，解绑事件监听器等。
