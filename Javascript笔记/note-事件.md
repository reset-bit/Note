[toc]

# js事件

## 事件绑定
### html事件绑定
在标签里面绑定事件函数
缺点：js和html混编、耦合

```
<div onclick='clickHandler'></div>// onclick=null解绑
function clickHandler() {
	console.log('you are clicked');
};
```

### dom 0级事件绑定
通过dom操作找到dom元素，进行事件绑定
缺点：多次绑定事件函数将覆盖
优点：js和html分离；兼容性好

```javascript
document.querySelector('.target').onclick = function() {
	console.log('you are clicked');
};
```

### dom 2级事件绑定
缺点：兼容性稍差，IE6/7/8不支持
优点：js和html分离；多次绑定事件函数不会覆盖；支持事件捕获阶段的事件绑定
> 事件生命周期：事件捕获阶段 - 事件源对象处理阶段 - 事件冒泡阶段
> 事件捕获从外向里；事件冒泡从里向外

```javascript
document.querySelector('.target').addEventListener('click', function() {
	console.log('you are clicked');
}(, true));
removeEventListener();// 必须与addEventListener使用相同的函数（使用变量保存）

// IE 6/7/8
document.querySelector('.target').attachEvent('click', function() {
	console.log('you are clicked');
});
detachEvent();
```

## 事件对象获取
```javascript
e = e || window.event;// 非IE 6/7/8事件处理函数的第一个参数；IE 6/7/8
// 事件源对象获取
e.target// 非IE 6/7/8
e.srcElement// IE 6/7/8
```

## 阻止事件
### 冒泡
```javascript
e.stopPropogation();// 非IE 6/7/8
e.cancelBubble = true;// IE 6/7/8
```
### 默认行为
如a标签跳转；表单submit提交
```javascript
e.preventDefault();// 非IE 6/7/8
return false;// IE 6/7/8，阻止冒泡+默认
```

## 自定义事件

```js
// 事件监听与移除监听
document.addEventListener('loaded', foo, false);
window.onunload = () => { document.removeEventListener('loaded', foo, false); }

// 不传参
function foo() {...}
const loaded = new Event('loaded');
document.dispatchEvent(loaded);

// 传参
function foo(e) { console.log(e.detail); }
const loaded = new CustomEvent('loaded', {'detail':{'arg':1}});
document.dispatchEvent(loaded);
```

## 事件循环

js中每行代码就是一个任务，马上执行完成的任务称**同步任务**，如声明变量、变量赋值等。无法马上执行完成的、需要消耗时间的任务称**异步任务**，如ajax请求数据、`setInterval`等。

js执行特点是：执行命令发出后，（若当前任务需要耗时）不会等待耗时任务执行完成，会继续往下执行。即，**js是无阻塞的**。

同步任务在同步任务队列中，异步任务放在异步任务队列中。只有当同步任务队列清空后，才会执行异步任务队列中的任务，反复循环。

异步任务队列分两种：**宏任务**`Ma/a/cro Task`队列与**微任务**`Mi/ai/cro Task`队列。先执行微任务，再执行宏任务。清空微任务之后执行宏任务，每次开启宏任务时查看微任务队列是否为空。在微任务执行时新添加的微任务不算在当前批次微任务。

> 宏任务：`setTimeout() / setInterval() / setImmediate() / <script>`、`ajax`回调函数、事件触发的回调函数
>
> 微任务：`then`开启的任务、`await`语句下面的任务（非同行）、`nextTick`开启的任务（在该时间片排首位，同步任务一个时间片，微任务一个时间片...将回调函数延迟在下一次dom更新数据后调用）、`Object.observer()`
>
> 1. `new Promise()`提供的**参数函数**会在当前时间片立即执行，参数函数中的代码算同步代码；
> 2. `await`关键字同行紧跟的任务算同步任务，在当前时间片下执行

```javascript
console.log(1);

setTimeout(() => {
    console.log(2);
    process.nextTick(() => consoel.log(3));
    new Promise(resove => {
        console.log(4);
        resolve();// 马上执行then()
    }).then(() => console.log(5));
}, 0);// 0ms后放入异步（宏任务）队列

new Promise(resolve => {
    console.log(7);
    resove();
}).then(() => console.log(8));

process.nextTick(() => console.log(6));

setTimeout(() => {
    console.log(9);
    process.nextTick(() => console.log(10));
}, 0);

new Promise(resolve => {
    console.log(11);
    resolve();
}).then(() => console.log(12));

// 1 7 11 6 8 12 2 4 3 5 9 10
```
