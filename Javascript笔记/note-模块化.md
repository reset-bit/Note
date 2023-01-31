[toc]

# 模块

实现特定功能的一组属性和方法的封装。
独立性是模块的重要特点，模块内部最好不与程序的其他部分直接交互。
为了在模块内部调用全局变量，必须显式地将其他变量输入模块。

```javascript
var module = (function ($, YAHOO){
	var count = 0;//私有变量，外部代码无法访问
	var m1 = function(){...};
	var m2 = function(){...};
	return {
		m1:m1,
		m2:m2
	};
})(jQuery, YAHOO);//IIFE

//模块添加方法
module = (function ($, YAHOO, mod){//将自身作为参数传入，修改后返回
	mod.m3 = function(){...};
	return mod;
})(jQuery, YAHOO, module || {});//宽放大模式：IIFE参数可以为空（防止第一个执行部分加载一个不存在的空对象）
```

# 模块化导入导出

ES6认为，所有js文件都是一个独立的模块，可以导出任意东西，怎么导出决定怎么导入。

## 导出

同一个变量可以同时使用`export`和`export default`导出。

`export:`导入时要加{}，可以出现在模块的任何位置，但必须处于模块顶层。一个模块可以`export`任意次。

```javascript
// 默认情况下，声明同时必须导出+不支持匿名
// 导出单个函数或变量
export let a = 100;
export function init() {...};
// 已声明变量，导出全部
let num = 1;
let str = 'zhangsan';
export {
	a,
    str
};
```

`export default`：一个模块只能有一个`export default`。导入时不需要加{}，可以使用任意变量名来接收。仅在第一次导出时运行代码，其他直接返回导出结果。

```javascript
export default 100;
```

## 导入

`import`必须在模块顶层，会提升到整个模块头部，首先执行。

单例模式：多次执行同一句import语句，只会执行一次。

赋值别名：`import {action as addressAction} from '../index.js'`。

如果导入css文件，可不加`from`。

```js
// 导入指定函数或对象
import { num, str } from 'xxx.js';
console.log(num, str);
// 导入全部
import all from 'xxx.js'
console.log(a);

// export default
import n from 'xxx.js';

// ES2020提案引入import()函数，支持动态加载模块，返回一个promise对象。类似node require()，但import为异步加载，require为同步加载
import('./xxx.js').then(...).catch(...);

// require
let imgUrl = require('../images/a.png');
```

> 1. 使用`require()`导入时，动态导入不可写成`let url; require(url);`（运行时变量导致webpack无法编译）。可写成`let url; require('../'+url);`。

## 导入导出复合写法

如果在一个模块之中先输入后输出同一模块/模块改名与整体输出/具名接口改为默认接口，`import`和`export`语句可以写在一起。

```js
// case 1
export { foo } from './xxx';

// case 2
export { foo as myFoo } from './xxx';
export * from './xxx.js';

// case 3
export { foo as default } from './xxx';
```

