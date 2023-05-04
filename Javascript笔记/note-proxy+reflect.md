[toc]

# 写在前面

ES6中，proxy和reflect支持允许拦截并定义基本语言的自定义行为，借助这两个对象，能够在JavaScript语言层面进行元编程



# Proxy

代理是一种设计模式，允许在访问对象的同时，添加一些额外的操作。代理对象与被代理对象实现相同的接口，代理对象将接收并处理所有对被代理对象的访问请求

proxy对象用于创建一个对象的代理，以监听一个对象的相关操作，实际是在访问对象之前进行部分拦截操作

创建一个新的proxy对象需要传入2个参数：被代理对象、捕获器。捕获器只能代理对象的外层属性，递归代理可监听值为对象的属性

> vue 2.0使用`Object.defineProperty()`实现MVVM双向数据监听，vue 3.0使用proxy重构，支持对象及新增属性监听
>
> `Object.defineProperty()`与proxy对象的区别是，前者只能监听对象的某一属性，后者可以监听整个对象（包括数组）

```js
const object = {
    name: 'o1',
    type: 'object',
    childs: [{
        name: 'ch1'
    }],
};
const handler = {
    get: function(target, key) {
        console.log('catch key', key);
        if(typeof target[key] === 'object') {
            return new Proxy(target[key], handler);// 递归代理
        }
        return target[key];
    },
    set: function(target, key, newVal) {
        if(key === 'type') {
            throw new Error('该属性不允许更改');
        }
        return true;
    },
};
const proxy = new Proxy(object, handler);
console.log(proxy.name);// catch key name / o1
proxy.type = 'function';// error
console.log(proxy.childs[0].name);// catch key childs / catch key 0/ catch key name / ch1

// 创建可撤销的proxy对象
const revocable = proxy.revocable(object, handler);
const revocableProxy = revocable.proxy;
// do something
revocable.revoke();
revocable.name;// TypeError
```

proxy对象钩子如下：

| 对象中的方法                         | 对应触发条件                                                 |
| ------------------------------------ | ------------------------------------------------------------ |
| `handler.getPrototypeOf()`           | `Object.getPrototypeOf` 方法的捕捉器                         |
| `handler.setPrototypeOf()`           | `Object.setPrototypeOf` 方法的捕捉器                         |
| `handler.isExtensible()`             | `Object.isExtensible` 方法的捕捉器                           |
| `handler.preventExtensions()`        | `Object.preventExtensions` 方法的捕捉器                      |
| `handler.getOwnPropertyDescriptor()` | `Object.getOwnPropertyDescriptor` 方法的捕捉器               |
| `handler.defineProperty()`           | `Object.defineProperty` 方法的捕捉器                         |
| `handler.has()`                      | `in` 操作符的捕捉器                                          |
| `handler.get()`                      | 属性读取操作的捕捉器                                         |
| `handler.set()`                      | 属性设置操作的捕捉器                                         |
| `handler.deleteProperty()`           | `delete` 操作符的捕捉器                                      |
| `handler.ownKeys()`                  | `Object.getOwnPropertyNames` 方法和 `Object.getOwnPropertySymbols` 方法的捕捉器 |
| `handler.apply()`                    | 函数被`apply`调用操作的捕捉器                                |
| `handler.construct()`                | `new` 操作符的捕捉器                                         |



# Reflect

reflect对象不同于proxy，它的所有属性和方法都是静态的，主要是为了弥补Object对象的一些缺陷

在早期，类似in、delete等操作符都放到了Object对象上，但Object作为一个构造函数，这些方法放到它身上并不合适。因此出现了reflect对象，将点语法规范成函数行为

reflect对象和proxy api一一对应，经常使用proxy拦截，使用reflect进行操作

reflect对象中的方法和object对象中的方法存在一些细微差别，reflect对象方法的返回值设置的更加合理。如`defineProperty()`，当对象属性定义失败时，reflect对象与成功一致返回boolean，object对象抛出typeError

```js
const object = {
    name: 'o1'
};
const handler = {
    get: function(target, key, receiver) {
        return Reflect.get(target, key, receiver);// receiver为proxy对象
    },
    set: function(target, key, newVal, receiver) {
        return Reflect.set(target, key, newVal, receiver);
    },
};

// reflect & object.defineProperty
'use strict';
const obj = {};
Object.defineProperty(obj, 'str', {
    value: 'string',
    writable: false,
});
obj.str = 'object string';// throw error
console.log(Reflect.set(obj, 'str', 'reflect string'));// false
console.log(obj);// {str: 'string'}
```

