## 1. 使用 ESLint 的 React Hooks 插件
- 尽管 `React` 官方文档给出了两条 [Hook 规则](https://zh-hans.reactjs.org/docs/hooks-rules.html)，但无论是新手还是经验丰富的 React 开发人员，都常常会忘记遵循 `React Hooks` 的规则。
- 因此，`React` 团队开发了一个名为 `eslint-plugin-react-hooks` 的 `ESLint` 插件，以帮助开发人员在自己的项目中以正确的方式编写 React Hooks。
- 这个插件能够帮助我们在尝试运行应用程序之前捕获并修复 Hooks 错误，所以最好将此插件添加到我们有使用 React Hooks 的项目中。
- 需要注意的是，`eslint-plugin-react-hooks` 插件约定，当在以大驼峰法命名的函数（视作一个组件）或在 `useSomething` 函数（视作一个自定义 Hook）中调用Hooks 时，`lint` 规则才能正常地工作：
![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c6bbb98a27484e2096449d0fba512a42~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)
- 如以下代码所示： 
![image](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af4891821307498ea8414687449fc69f~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)
- 当我们在项目中使用了 eslint-plugin-react-hooks 插件后，会发生 eslint 报错，提示我们不能条件式调用 Hooks：
![image](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/50f020dabddc461a81f23df07fd753d5~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)
- 而当我们将组件改成匿名默认导出时, 可以看到，lint 规则并没有生效（[相关issue](https://github.com/facebook/react/issues/19155)）：
![image](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/454788e9f6b14dc0bd0975fd96e4f32d~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)
- 所以在实际开发过程中，在写函数式组件时，要给组件进行大驼峰法命名；而在写自定义 `Hook` 时，也要遵循 `useSomething` 的命名规则

## 2. 以正确的顺序创建函数组件
- 当创建类组件时，遵循一定的顺序可以帮助我们更好地维护和改进 React 应用程序代码：
  - `static` 开头的类属性
  - 构造函数，`constructor`
  - `getter/setter`
  - 组件生命周期
  - `_ `开头的私有方法
  事件监听方法，`handle*`
  `render*`开头的方法，有时候 render() 方法里面的内容会分开到不同函数里面进行，这些函数都以 render* 开头
  `render()` 方法


- 虽然不按上面顺序组织代码，组件功能也不会有什么影响。但是，如果所有的组件都按这种顺序来编写，那么维护起来就会方便很多，多人协作的时候别人理解代码也会一目了然。
- 对函数组件而言，并没有所谓的构造器和生命周期函数，只要编写的代码按照 [Hook 规则](https://zh-hans.reactjs.org/docs/hooks-rules.html)实施，一般都不会遇到什么问题：
![image](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/170227c789dc4c67951456ccb0aed182~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)
- 与类组件一样，虽然编写顺序对功能实现没有大的影响，但是，为函数组件创建定义的结构能够改善项目的可读性。
- 推荐在函数顶部使用 `useState Hook` 和 `useRef Hook`，然后使用 `useEffect Hook` 编写订阅，接着编写与组件作业相关的其他函数，最后返回要由浏览器渲染的元素：
![image](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b9498fd18fc74e1d9cc7c97c4d931ca3~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)


## 3.  掌握useEffect中的异步用法
- 我们都知道，`useEffect` 用来引入具有副作用的操作，最常见的就是向服务器请求数据。对 React Hook 不熟悉的小白可能会在没有使用 `Typescript` 的项目中犯以下错误：
![image](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2e5270745dd4fc68c23a9c1998168e2~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)
- 当在跳转路由（离开当前界面）的时候会遇到如下错误：
![image](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39225b82537c4a319870bbeac910b4de~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)
- 而我们都知道，Typescript 能帮我们做静态类型检查，若在使用 Typescript 的项目的 `useEffect` 中错误使用了异步函数，编译器就会及时地报出下面的错误（而不至于在组件卸载时才触发报错）：
> Argument of type '() => Promise' is not assignable to parameter of type 'EffectCallback'.
- 发生这样错误的原因是使用异步函数导致我们在使用 `useEffect()` 的时候返回的是一个值而非函数。通常，`effects` 需要在组件销毁之前清除创建的资源，例如订阅或计时器ID。为此，传递给 useEffect的函数应该返回一个清理函数。原本需要返回的是一个`cleanup` 函数，而使用异步函数会使 `callback` 返回 `Promise` 而不是 cleanup 函数，因此当组件销毁时候，就会发生报错的情况。
- 为了避免直接将 `Promise` 作为 `effects` 返回值，我们可以把异步函数独立出来：
![image](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/46ec8ded987245bebfc94baed2202d47~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)
- 或使用IIFE( Immediately Invoked Function Expression)匿名函数表达式：
![image](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/10385e7f4b254d918d014f1de5703e93~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)


## 4. 尽量避免使用 useLayoutEffect
- `useEffect` 和 `useLayoutEffect`，是两个工作方式很相似的`React Hook`，它们二者的不同在于执行时机：
  - useEffect 是在渲染函数执行完成，并绘制到屏幕之后，再异步执行
  - useLayoutEffect是在渲染函数执行之后，屏幕重绘前同步执行
- 因为 `useLayoutEffect` 是同步执行的，因此会发生阻塞，直到该 `effect` 执行完成才会进行页面重绘，如果 effect 内部有执行很慢的代码，可能会引起性能问题。因此，React 官方指出，尽可能使用标准的 useEffect 以避免阻塞视觉更新。
- `useLayoutEffect` 也不是毫无作用，下面介绍它的一个使用场景。
![image](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47d8b31e14a740acb90f261b57949490~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)
- 当点击按钮时，会发现页面发生闪烁： ![image](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fd56e1d181874c16a074db4bb5047b92~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)
- 原因在于，`useEffect` 的触发时机会被延迟到 DOM 绘制完成。因此我们点击按钮 `setState` 后，其实经历了 设置高度为 50 -> 绘制屏幕 -> 设置高度为100 的两次 render 过程，所以肉眼才会看到中间过渡的 state 导致的闪烁的感觉。
- 而前面提到，`useLayoutEffect` 会在所有 DOM 改变后，同步调用。在浏览器运行绘制之前，useLayoutEffect 内部的更新将被同步刷新。正因为这个 hook 的特性，我们可以使用它来让 DOM 的渲染慢一拍，等待 state 真正更新完后才去渲染浏览器的画面。
- 我们将 useEffect 改为 useLayoutEffect：
![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3bed8d8e7a7b4388a6b886dd0947d937~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

## 5. 善用 useMemo / useCallback
- 前面提到，我们可以使用 `React.memo()` 这个高阶组件来控制函数组件的重复渲染。
- 导致重复渲染的原因是 `React Hooks` 使用的是函数组件，父组件的任何一次修改，都会导致子组件的函数执行，从而重新进行渲染。
- 因此，为了性能方面考虑，除了使用 `React.memo()` 对函数组件进行包装，我们还可以使用 React 提供的 `useCallback` 和 `useMemo` 来对针对函数和函数的返回值进行缓存。
- 需要注意的是， useCallback 和 useMemo 要结合 React.memo() 才能避免子组件无效渲染。
