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