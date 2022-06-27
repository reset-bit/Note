[toc]

# Object相关方法

```javascript
xxx.hasOwnProperty();// 返回值为boolean，指示对象自身属性中是否具有指定的属性
Object.getOwnPropertyNames(xxx);// xxx的键名数组
// 直接在对象上定义一个新属性，或者修改已有属性。返回值为新对象
Object.defineProperty(obj, propertyName, {
    value: '',
    writable: false | true,// 是否可修改，严格模式下再修改
    enumerable: false | true,
    configurable: false | true,// 是否可配置，配置为false之后再配置会报错
    set: function() {},
    get: function() {}
});

// 对+0与-0、NaN与NaN会返回正确结果。并不是比===和==严格，只是有更特殊的用途
Object.is(val1, val2);

// 伪深拷贝，另建地址拷贝，仅拷贝一层。returnTarget === target，合并source到target并返回
let returnTarget = Object.assign(target, source);

let newObj = Object.create(obj);// 返回一个新对象，令该对象的__proto__为obj

let keyArr = object.key
```

- `new Object()`与`Object.create()`的区别：`new Object()`通过构造函数来创建对象，添加的属性在自身实例下。`Object.create()`返回一个__ proto __为指定对象的新对象。

  ```javascript
  // new Object()
  let obj1 = {a:1};
  let obj2 = new Object(obj1);
  console.log(obj2);// {a:1}
  console.log(obj2.__proto__);// {}
  console.log(obj2.a);// 1
  
  // Object.create()
  let obj1 = {a:1};
  let obj2 = Object.create(obj1);
  console.log(obj2);// {}
  console.log(obj2.__proto__);// {a:1}
  console.log(obj2.a);// 1
  ```

- `new Object()`与`{}`的区别：`{}`是对象字面量。如果`new Object()`中没有传入参数，与`{}`相同。传入`String`返回`String`，类似`new String()`。传入`Number`同理。**字面量可以立即求值，而`new Object()`本质上是方法，过程中涉及到堆栈调用及释放。故字面量赋值比`new Object()`高效。**

> **字面量**：不用new操作符来创建实例的、一种简单易阅读的方法。包括字符串字面量、数组字面量、对象字面量、函数字面量。

- `new String()`与‘’的区别：
	`let s = 'text';`s是一个原始数据类型，没有方法，只是一个指向原始数据存储器引用的指针，变量存储在**常量池**中，而`new String()`变量存储在**堆**中。因此相较于`let s = new String('text');`有**更快的随机访问速度**。当调用`s.charAt(i)`时，js会对其进行自动包装。对`typeof s`来说，更准确的是`s.valueOf(s).prototype.toString.call = [object String]`。**自动装箱行为**会根据需要进行来回包装类型，但标准操作速度很快。
	
	如果想强制自动装箱或将原始类型转换为包装类型，可以使用`Object.prototype.valueOf()`
    ```javascript
  let a = 'some';
  console.log(typeof a);// string，
	
  let a = new String('some');
  console.log(typeof a);// object，String对象
	
  let a = new Object('some');
  console.log(typeof a);// object，String对象
  
	  ```
	
- 

## 判断两个对象内容相等

```javascript
// 递归实现
function isObjectValueEqual(a, b) {
    let aProps = Object.getOwnPropertyNames(a);
    let bProps = Object.getOwnPropertyNames(b);
    if(aProps.length !=== bProps.length) return false;
    for(let i = 0; i < aProps.length; ++i) {
        let propName = aProps[i];
        // 若b没有该属性值
        if(!b.hasOwnProperty(propName)) return false;
        // 若当前比较属性值是对象，并且递归之后返回false
        if(a[propName] instanceof Object && !isObjectValueEqual(a[propName], b[propName])) return false;
    }
    return true;
}
```

## 判断简单类型的数组内容相等

```javascript
// 1.
JSON.stringify(arr1.sort()) === JSON.stringify(arr2.sort())
// 2.
arr1.sort().toString() === arr2.sort().toString()
```



# 数据结构

## Map(es6)
键值对**数组**
```javascript
let map = new Map([
    [1, 'one'],
    [2, 'two'],
    [3, 'three']
]);
map.set('a', 'aa');
map.set('b', 'bb');
map.get('a');// 'aa'
map.delete('a');
map.has('a');// false
map.clear();// null
map.forEach(function(item, key, obj) {
    console.log(item.toString());
});
// 按值排序
let arr = Array.from(map);// 将map转为数组， 也可[...map]
arr.sort(function(a, b) { return a[1] - b[1]; });// a[1]=>a.value
console.log(map);
// 获取keys并转为数组
let keys = [...map.keys()];
// 获取values并转为数组
let values = [...map.values()];
```

## Set(es6)

元素不重复的键值对数组，支持存放NaN、undefined

```javascript
let set = new Set([1, 2, 3, 3, 4]);//[1,2,3,4]
// let set = new Set('Hello');//['H','e','l','l','o']
set.add('some');//[1,2,3,4,"some"]
set.has(3);
set.size;//5
set.delete('some');
set.forEach(function(value) {
    console.log(value);
});
for(let i of set.keys()) {
    console.log(i);
}
for(let [key, value] of set.entries()) {
    console.log(key);
}
// 求交
let intersection = new Set([...set1].filter(x => set2.has(x)));
// 求差
let difference = new Set([...set1].filter(x => !set2.has(x)));
// 数组去重
let numbers = [1,2,1,2,1,4,3];
console.log([...new Set(numbers)]);//[1,2,3,4]
```

## 二叉树

```js
// 递归遍历
function traverse(root) {
    if(root !== null) {
        // 前序：根左右
        console.log(root.val);
       	traverse(root.left);
        traverse(root.right);
        // 中序：左根右
       	traverse(root.left);
        console.log(root.val);
        traverse(root.right);
        // 后序：左右根
       	traverse(root.left);
        traverse(root.right);
        console.log(root.val);
    }
};
// 非递归遍历
function traverse(root) {
    const stack = [], res = [];
    // 前序
    root && stack.push(root);
    while(stack.length > 0) {
        let cur = stack.pop();
        res.push(cur.val);
        cur.right && stack.push(cur.right);// 先入栈的后出栈，所以先入右子树
        cur.left && stack.push(cur.left);
    }
    // 中序
    while(root || stack.length) {
        if(root.left) {// 迭代左子树，令左子树依次入栈
            stack.push(root);
            root = root.left;
        } else if(root.right) {// 若左子树为空，（第一次）遍历到第一个叶子结点。输出当前节点val，迭代右子树
            res.push(root.val);
            root = root.right;
        } else if(!root.left && !root.right) {// 若左右子树都为空，则输出当前节点，弹出栈顶，返回上一层。注意root.left置空，防止再次进入循环
            res.push(root.val);
            root = stack.pop();
            root && (root.left = null);
        }
    }
    // 后序
    while(root || stack.length) {
        if(root.left) {// 迭代左子树，令左子树依次入栈
            stack.push(root);
            root = root.left;
        } else if(root.right) {// 若左子树为空，遍历到叶子节点，迭代右子树
            stack.push(root);
            root = root.right;
        } else {// 若左右子树都为空，输出当前节点，弹出栈顶，返回上一层。注意root.left、root.right置空
            res.push(root.val);
            root = stack.pop();
            if(root && root.left) {
                root.left = null;
            } else if(root && root.right) {
                root.right = null;
            }
        }
    }
    return res;
};
```

