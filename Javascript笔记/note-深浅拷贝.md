[toc]

# 深浅拷贝

引用对象真实数据存放在堆内存，栈中存放该对象的引用。

深拷贝和浅拷贝只针对`Object`和`Array`这样的引用数据类型。

> 浅拷贝只赋值某个对象的引用，而不复制对象本身，新旧对象还是共享同一块内存。
>
> 深拷贝会创造一个一模一样的对象，新对象跟源对象不共享内存，修改新对象不会修改源对象。

## 浅拷贝

### 对象

```js
newObj = {.};
Object.assign(newObj, obj);
newObj = Object.create(obj);
```

### 数组

```js
newArr = [...arr];
newArr = arr.map(x => x);
newArr = arr.filter(() => true);
newArr = arr.slice();
newArr = arr.concat();
newArr = Array.from(arr);// Array.from()将任何可迭代对象转化为数组
newArr = arr.copyWithin(0);// es6
```

## 深拷贝

- `let newArr = JSON.parse(JSON.stringify(arr))`。能处理对象和数组，不能处理函数
- `let newArr = $.extend(true, [], arr)`

- 手写递归方法

  ```js
  function deepClone(obj) {
      let objClone = Array.isArray(obj) ? [] : {};
      for(let key in obj) {
          let value = obj[i];
          if(checkType(value) === 'Object' || checkType(value) === 'Array') {
              objClone[i] = deepClone(value);
          } else {
              objClone[i] = value;
          }
      }
      return objClone;
  }
  function checkType(target) {
      return Object.prototype.toString.call(target).slice(8, -1);
  }
  ```

