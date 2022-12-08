# React基础

1. react根组建开启React.StrictMode严格模式
2. 避免在元素直接定义ref，需通过createRef创建
3. setState同步还是异步
  - setState在同步逻辑中，异步更新状态和dom
  - setState在异步逻辑中，同步更新状态和dom
  - setState接收第二个参数，是回调函数，状态和dom更新完后触发
4. input表单在react中和原生的区别：value受控，defaultValue非受控。onInput和onChange等效

##  组件通信的方式
1. 父子组件通信方式