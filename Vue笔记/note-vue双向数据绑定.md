# vue2双向数据绑定/响应式/观测机制

![image-20211011142726425](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20211011142726425.png)

`data->dom`：

- `observer`将原生数据对象调用**Object.defineProperty()**处理，令其改造成**可观察对象**，进行数据劫持。主要是在原生数据对象中增加set、get方法。在set()中发布emit事件，通知watcher发生数据改变。

- 在`watcher`求值，即模板渲染过程中，每一个被取值的可观察对象将成为`watcher`的一个依赖`dep(dependency)`。每个**dep实例**都可以**记录依赖并派发更新**。

- `compiler`解析模板中的指令，在模板编译时**订阅数据**，订阅注册on事件并指定回调函数，在回调函数中改变dom。

  > compiler模板渲染参考【vue渲染过程+虚拟dom】

`dom->data`：`compiler`中直接监听onInput事件或触发emit事件，通知watcher发生数据改变。

> `$emit(eventName[, args])`：触发事件
>
> `$on(eventName, callback)`：监听事件

## 调度器scheduler

调度器维护一个执行队列，该队列中同一个watcher仅会存在一次，避免函数重复频繁运行。

队列中的watcher不是立即执行，而是通过nextTick工具方法将需要执行的watcher放到事件循环的微队列中。也就是说，**当响应式数据变化时，render()的执行是在异步微队列中的。**

> `Vue.$nextTick(callback);`：回调函数将在dom更新完成后被调用，相当setTimeout()17s左右。组件内this自动绑定到当前vue实例上，故可直接调用`this.$nextTick()`。返回一个Promise对象，支持async
>
> ```js
> methods: {
>     updateMessage: function() {
>         this.message = '已更新'；
>         console.log(this.message);// '未更新'
>         this.$nextTick(() => console.log(this.message));// '已更新'
>     }
>     // or
>     updateMessgae: async function() {
>         this.message = '已更新';
>         await this.$nextTick();
>         console.log(this.message);// '已更新'
>     }
> }
> ```
>
> 

## 观察者模式 / 发布-订阅模式

```js
// watcher实现
// 事件仅在观察者emit事件时调用，on标记需要执行的逻辑
let event = {
    obj: {},// obj: { classover: [function1, function2...] }
    on: function(eventName, callback) {
        if(!eventName in this.obj) { this.obj[eventName] = []; }
        this.obj[eventName].push(callback);
    },
    emit: function(eventName, ...args) {
        if(eventName in this.obj) {
            for(callback of this.obj[eventName]) {
                callback(...args);// 调用回调函数
            }
        }
    }
};

event.on('classover', function(...args) {// 订阅者
    console.log('i know class is over');
});
event.emit('classover', 'real');// 观察者
```

## Object.defineProperty()与装饰器模式

允许向一个现有的对象添加一个新的功能，同时不改变其结构。

```js
let obj = {};
obj.num = null;
Object.defineProperty(obj, num, {
    setter: function(value) {
        this.num = value;
        // todo...
    },
    getter: function() { return this.a; }
});

// 监听数组元素改变
obj.arr = [];
const methods = ['push', 'pop', 'unshift', 'shift', 'sort', 'reverse'];
const vueArrayPrototype = {};
for(let method of methods) {
    vueArrayPrototype[method] = function(...args) {
        ArrayPrototype[method].apply(this, args);
        // todo...
    };
}
arr._proto__ = vueArrayPrototype;
```

