# hash模式与history模式

vue是SPA，保证只有一个html页面，**与用户交互时，不刷新和跳转页面的同时，为SPA中每个视图匹配一个特殊的URL**。在刷新、前进、后退和SEO时均通过这个特殊的URL实现。

需要满足：

- 可以监听到URL变化
- 改变URL，且不让浏览器发送请求、不刷新页面


## hash模式

在URL后面加上#，如`http://127.0.0.1:5500/前端路由/hash.html#/page1`，`#/page1`即为hash值

- `location.hash`可以获取hash值 
- `hashchange()`是hash值发生改变时调用的函数

优点：

- 兼容性好
- 处理资源加载和ajax请求外，不会发起其他请求
- 不需要服务器端进行任何设置和开发

缺点：

- 对需要使用锚点功能的需求会与目前路由机制冲突
- 对部分需要重定向的操作，后端无法获取hash部分内容，导致后端无法获取URL中的数据

### window.location对象

改变其属性值修改页面url。

SPA改变URL不刷新页面可以：

- `location.href`赋值时只改变URL hash
- 直接赋值`location.hash`

```js
location.href
location.hash
location.search// 直接刷新页面
location.pathname
```

## history模式

H5新增的`history`接口允许操作浏览器曾经在标签页或框架里访问的会话历史记录

基本api：

```js
history.go(n)// 路由跳转几步，支持正负值
history.back()// 路由后退，可点击后退按钮模拟此方法
history.forward()// 路由前进，可点击前进按钮模拟此方法
history.pushState(state, title, url)// 添加一条路由历史记录，不触发跳转
history.popstate()// 当活动历史记录条目更改时，将触发本事件
```

优点：

- 重定向中不会丢失URL中参数
- 可以使用`history.state`获取当前URL对应的状态信息

缺点：

- 兼容性不如hash路由
- 需要后端支持，每次返回html文档

## 二者对比

形式上，hash模式url里总是带着#号，开发中默认使用这个模式。如果需要考虑URL规范性，则需要使用history模式。因为该模式URL不带#，适合推广宣传。

功能上，有的app分享第三方链接时不允许带#，需要使用history模式。但是history模式在访问二级页面（vue router配置children节点）的时候做刷新操作，会报404，需要和后端配合重定向首页路由。如果后端没有正确的配置，需要前端自己配置404页面。

## vue-router

```js
const router = new VueRouter({
    mode: 'history',
    routes: [
        { path: '*', component: NotFoundComponent }
    ]
});
```

## 给SPA做SEO

1. SSR`server-side rendering`服务器渲染：将组件或页面通过服务器生成html，再返回给浏览器
2. 静态化：通过程序将动态页面抓取并保存为静态页面
3. 针对爬虫处理