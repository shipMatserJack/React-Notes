首先阅读
https://www.jianshu.com/p/947b7b1e3bba接着阅读
https://www.jianshu.com/p/3eccc6902a3a补充API
https://react-query-v3.tanstack.com/reference/useQuery中文使用手册
https://haofly.net/react-query/
Next.js场景
数据在服务端保存在内存中，通过水合（hydration）在客户端使用
https://react-query-v3.tanstack.com/guides/ssr答疑
cache存储在哪里？
context存储，目前也支持createWebStoragePersistor (Experimental) | React Query | TanStack
为什么不推荐修改默认的缓存行为
在上文中解释了staleTime（新鲜度或是否过时，默认0s）和cacheTime（缓存存放时间，默认5min）的概念。
在默认配置下，第一次肯定会请求，第二次触发该请求，会立即使用缓存数据返回并在后台发起请求来更新缓存。
好处是：既使用缓存快速呈现页面，又确保可以使用最新的数据
https://react-query-v3.tanstack.com/guides/cachinguseMutation
1. 修改缓存
2. 创建请求对象，有别于hook监听依赖变化触发请求，该方法是创建对象用于主动请求
function App() {
   const mutation = useMutation(newTodo => {
     return axios.post('/todos', newTodo)
   });
  return (
  <button
    onClick={() => {
      mutation.mutate({ id: new Date(), title: 'Do Laundry' })
     }}
   >
     Create Todo
   </button>
  );
}
https://www.jianshu.com/p/caca0e212220核心探究
https://juejin.cn/column/7134900037124882439https://juejin.cn/post/7176576611012050999
QueryClient, QueryCache, Query, QueryObserver
1. 当我们初始化React项目时,会先全局生成一个QueryClient,用于提供封装好的请求方法,并使用React.context分享给App下所有的组件
2. QueryClient上会挂载一个QueryCache,用来保存和管理所有的Query
3. Query用于保存管理数据和请求状态,每个请求回来的数据都会new一个Query并保存在上面
4. 每个useQuery钩子,都会给组件创建一个QueryObserver,用于监听Query中数据的变化,通知组件进行更新