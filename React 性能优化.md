# React 性能优化技巧

## 使用 React.memo 避免不必要的重渲染
对于函数组件，可以使用 React.memo 来避免在父组件重新渲染时，子组件的无意义渲染。它只会在组件的 props 发生变化时才重新渲染。
```js
export default React.memo(MyComponent);
```

## 使用懒加载 (React.lazy 和 Suspense)
分割代码并懒加载是提高应用加载速度的一个有效方法。可以使用动态 import() 语法加上 React.lazy 和 Suspense 来实现按需加载组件。
```js
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <React.Suspense fallback={<div>Loading...</div>}>
      <OtherComponent />
    </React.Suspense>
  );
}
```
## 使用 shouldComponentUpdate 或 React.PureComponent
对于类组件，可以利用 shouldComponentUpdate 生命周期方法来决定何时更新组件。如果你在做浅比较，使用 React.PureComponent 可以自动给你实现这一点。
```js
class MyComponent extends React.PureComponent {
  // ...
}
```

## 避免在渲染方法中创建新的对象或函数
在组件的渲染方法中创建新的对象或函数会在每次渲染时产生新的引用，从而可能会触发子组件的不必要渲染。应当将这些值移到组件外部或使用 useCallback 和 useMemo 钩子。
```js
function MyComponent({someProp}) {
  const memoizedValue = useMemo(() => computeExpensiveValue(someProp), [someProp]);
  const memoizedCallback = useCallback(() => {
    doSomething(someProp);
  }, [someProp]);
  
  // ...
}
```

## 减少不必要的重渲染
使用 useCallback 钩子来缓存回调函数，避免函数的不必要创建，使用 useMemo 钩子来缓存复杂计算的结果
```js
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);

const memoizedResult = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

## 窗口化或虚拟化长列表
如果列表非常长，使用如 react-window 或 react-virtualized 这样的库来实现窗口化或虚拟化的渲染可以显著提高性能。
```js
import { FixedSizeList as List } from 'react-window';

<List
  height={150}
  itemCount={1000}
  itemSize={35}
  width={300}
>
  {({ index, style }) => (
    <div style={style}>Row {index}</div>
  )}
</List>
```

## 避免内联样式
尽量使用 className 进行样式的声明。避免使用对象字面量作为 style 属性的值，因为这样会在每次渲染时创建一个新对象。
```jsx
<div className="my-div" />
```

## 对状态更新进行批处理
使用 setState 进行状体更新时，React 会将多个 setState 调用批处理成单次更新，这样可以避免不必要的渲染。
```js
this.setState((prevState) => {
  return { /* updates based on previous state */ };
});
```