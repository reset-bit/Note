# 浅拷贝和深拷贝

引用对象真实数据存放在堆内存，栈中存放该对象的引用。

深拷贝和浅拷贝只针对`Object`和`Array`这样的引用数据类型。

> 浅拷贝只赋值某个对象的引用，而不复制对象本身，新旧对象还是共享同一块内存。
>
> 深拷贝会创造一个一模一样的对象，新对象跟源对象不共享内存，修改新对象不会修改源对象。

## 浅拷贝

- `Object.assign({}, obj)`。当object只有一层时，是深拷贝
- `let newObj = Object.create(obj)`
- `newArr.concat(arr)`
- `let newArr = arr.slice(startPos, endPos)`

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

