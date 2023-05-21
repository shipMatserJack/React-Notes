
# React Hooks
使用hooks理由
1. 高阶组件为了复用，导致代码层级复杂
2. 生命周期的复杂
3. 写成functional组件,无状态组件 ，因为需要状态，又改成了class,成本高

## hooks 之数据更新驱动
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
**useReducer 基础介绍：** 

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

## hooks 之执行副作用
### useEffect
React hooks也提供了 api ，用于弥补函数组件没有生命周期的缺陷。其本质主要是运用了 hooks 里面的 useEffect ， useLayoutEffect，还有 useInsertionEffect。其中最常用的就是 useEffect 。我们首先来看一下 useEffect 的使用。

**useEffect 基础介绍：**
```js
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

### useLayoutEffect
**useLayoutEffect 基础介绍：**

useLayoutEffect 和 useEffect 不同的地方是采用了`同步执行`，那么和useEffect有什么区别呢？

① 首先 useLayoutEffect 是在 DOM 更新之后，浏览器绘制之前，这样可以方便修改 DOM，获取 DOM 信息，这样浏览器只会绘制一次，如果修改 DOM 布局放在 useEffect ，那 useEffect 执行是在浏览器绘制视图之后，接下来又改 DOM ，就可能会导致浏览器再次回流和重绘。而且由于两次绘制，视图上可能会造成闪现突兀的效果。  
② useLayoutEffect callback 中代码执行会阻塞浏览器绘制。

**useLayoutEffect 基础用法：**  
```js
const DemoUseLayoutEffect = () => {
    const target = useRef()
    useLayoutEffect(() => {
        /*我们需要在dom绘制之前，移动dom到制定位置*/
        const { x ,y } = getPositon() /* 获取要移动的 x,y坐标 */
        animate(target.current,{ x,y })
    }, []);
    return (
        <div >
            <span ref={ target } className="animate"></span>
        </div>
    )
}
```

## hooks 之状态派生与保存
### useMemo
useMemo 可以在函数组件 render 上下文中同步执行一个函数逻辑，这个函数的返回值可以作为一个新的状态缓存起来。那么这个 hooks 的作用就显而易见了：

场景一：在一些场景下，需要在函数组件中进行大量的逻辑计算，那么我们不期望每一次函数组件渲染都执行这些复杂的计算逻辑，所以就需要在 useMemo 的回调函数中执行这些逻辑，然后把得到的产物（计算结果）缓存起来就可以了。

场景二：React 在整个更新流程中，diff 起到了决定性的作用，比如 Context 中的 provider 通过 diff value 来判断是否更新

**useMemo 基础介绍：**
```js
const cacheSomething = useMemo(create,deps)
```
① create：第一个参数为一个函数，函数的返回值作为缓存值，如上 demo 中把 Children 对应的 element 对象，缓存起来。  
② deps： 第二个参数为一个数组，存放当前 useMemo 的依赖项，在函数组件下一次执行的时候，会对比 deps 依赖项里面的状态，是否有改变，如果有改变重新执行 create ，得到新的缓存值。  
③ cacheSomething：返回值，执行 create 的返回值。如果 deps 中有依赖项改变，返回的重新执行 create 产生的值，否则取上一次缓存值。

**useMemo 基础用法：**
1. 派生新状态：
   通过 useMemo 得到派生出来的新状态 contextValue ，只有 keeper 变化的时候，才改变 Provider 的 value 。
    ```js
    function Scope() {
        const keeper = useKeep()
        const { cacheDispatch, cacheList, hasAliveStatus } = keeper
      
        /* 通过 useMemo 得到派生出来的新状态 contextValue  */
        const contextValue = useMemo(() => {
            return {
                cacheDispatch: cacheDispatch.bind(keeper),
                hasAliveStatus: hasAliveStatus.bind(keeper),
                cacheDestory: (payload) => cacheDispatch.call(keeper, { type: ACTION_DESTORY, payload })
            }
          
        }, [keeper])
        return <KeepaliveContext.Provider value={contextValue}>
        </KeepaliveContext.Provider>
    }
    ```
2. 缓存计算结果：
    ```js
    function Scope(){
      const style = useMemo(()=>{
        let computedStyle = {}
        // 经过大量的计算
        return computedStyle
      },[])
      return <div style={style} ></div>
    }
    ```

3. 缓存组件,减少子组件 rerender 次数：
    ```js
    function Scope ({ children }){
      const renderChild = useMemo(()=>{ children()  },[ children ])
      return <div>{ renderChild } </div>
    }
    ```


### useCallback
**useCallback 基础介绍：**

`useMemo` 和 `useCallback` 接收的参数都是一样，都是在其依赖项发生变化后才执行，都是返回缓存的值，区别在于 useMemo 返回的是`函数运行的结果`，useCallback 返回的是`函数`，这个回调函数是经过处理后的也就是说父组件传递一个函数给子组件的时候，由于是无状态组件每一次都会重新生成新的 props 函数，这样就使得每一次传递给子组件的函数都发生了变化，这时候就会触发子组件的更新，这些更新是没有必要的，此时我们就可以通过 usecallback 来处理此函数，然后作为 props 传递给子组件。

**useCallback 基础用法：**
```js
/* 用react.memo */
const DemoChildren = React.memo((props)=>{
   /* 只有初始化的时候打印了 子组件更新 */
    console.log('子组件更新')
   useEffect(()=>{
       props.getInfo('子组件')
   },[])
   return <div>子组件</div>
})

const DemoUseCallback=({ id })=>{
    const [number, setNumber] = useState(1)
    /* 此时usecallback的第一参数 (sonName)=>{ console.log(sonName) }
     经过处理赋值给 getInfo */
    const getInfo  = useCallback((sonName)=>{
          console.log(sonName)
    },[id])
    return <div>
        {/* 点击按钮触发父组件更新 ，但是子组件没有更新 */}
        <button onClick={ ()=>setNumber(number+1) } >增加</button>
        <DemoChildren getInfo={getInfo} />
    </div>
}
```

## hooks 之状态获取与传递
### useContext
**useContext 基础介绍**  
可以使用 useContext ，来获取父级组件传递过来的 context 值，这个当前值就是最近的父级组件 Provider 设置的 value 值，useContext 参数一般是由 createContext 方式创建的 ,也可以父级上下文 context 传递的 ( 参数为 context )。useContext 可以代替 context.Consumer 来获取 Provider 中保存的 value 值
```jsx
const contextValue = useContext(context)
```
useContext 接受一个参数，一般都是 context 对象，返回值为 context 对象内部保存的 value 值。

**useContext 基础用法：**  
```jsx
/* 用useContext方式 */
const DemoContext = ()=> {
    const value:any = useContext(Context)
    /* my name is alien */
return <div> my name is { value.name }</div>
}

/* 用Context.Consumer 方式 */
const DemoContext1 = ()=>{
    return <Context.Consumer>
         {/*  my name is alien  */}
        { (value)=> <div> my name is { value.name }</div> }
    </Context.Consumer>
}

export default ()=>{
    return <div>
        <Context.Provider value={{ name:'alien' , age:18 }} >
            <DemoContext />
            <DemoContext1 />
        </Context.Provider>
    </div>
}
```

### useRef
**useRef基础介绍** 
useRef 可以用来获取元素，缓存状态，接受一个状态 initState 作为初始值，返回一个 ref 对象 cur, cur 上有一个 current 属性就是 ref 对象需要获取的内容。  
```jsx
const cur = React.useRef(initState)
console.log(cur.current)
```
**useRef 基础用法：**   
useRef 获取 `DOM 元素`，在 React Native 中虽然没有 DOM 元素，但是也能够获取组件的节点信息（ fiber 信息 ）。
```jsx
const DemoUseRef = ()=>{
    const dom= useRef(null)
    const handerSubmit = ()=>{
        /*  <div >表单组件</div>  dom 节点 */
        console.log(dom.current)
    }
    return <div>
        {/* ref 标记当前dom节点 */}
        <div ref={dom} >表单组件</div>
        <button onClick={()=>handerSubmit()} >提交</button> 
    </div>
}
```
`useRef 保存状态`， 可以利用 useRef 返回的 ref 对象来保存状态，只要当前组件不被销毁，那么状态就会一直存在。
```jsx
const status = useRef(false)
/* 改变状态 */
const handleChangeStatus = () => {
  status.current = true
}
```