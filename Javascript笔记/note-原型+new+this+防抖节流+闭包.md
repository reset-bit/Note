[toc]
# 原型和原型链
> 在 JavaScript 的语言特性中没有“类”的概念，为了便于理解，用【类】来称呼那些实际上将会调用构造函数的 Function 对象，用【对象实例】来称呼并强调通过调用该对象的构造函数而生成的对象。

> 某个【对象】的原型/原型对象就是所有该【对象实例】共享的一组属性和方法。这些属性和方法定义在这些对象实例的构造函数`prototype`属性中。`prototype`是一个指针，指向对象的原型对象。也可以通过实例对象的`__proto__(dunder proto)`属性获取构造函数原型对象。原型对象本身也可能有原型，并从中继承属性和方法，以此类推。这种关系被称为原型链。调用方法时，js引擎首先在当前对象上寻找是否具有该方法，有就调用，没有就将该请求委托给原型对象。如果在原型对象上也没有该方法，就委托给到原型对象的原型对象，以此类推。js中，根对象是`Object.prototype`，它的原型为null。**未指定原型对象的`function`原型对象为{}。**

> 关于\__proto\__和prototype：
> 1. 实例化对象有属性\__proto\__，指向该对象的构造函数的原型对象；
> 2. 方法除了有属性\__proto\__，还有属性prototype，prototype指向该方法的原型对象

> 使用new声明的对象创建过程：
> 1. 以构造器的`prototype`属性为原型创建新对象
>
> 2. 将创建新对象的this和构造函数需要的参数传给构造器，令其执行
>
> 3. 如果构造器没有返回对象，则返回第一步创建的对象
>
>    ```javascript
>    function myNew() {
>        let obj = new Object();
>        let constructor = Array.prototype.shift.call(arguments);
>        obj.__proto__ = constructor.prototype;// 关联原型对象
>        let ret = constructor.apply(obj, arguments);// 传递this和参数
>        return typeof ret === 'object' ? ret : obj;
>    };
>    ```
>

```javascript
var str = '12';
console.log(str.__proto__);// String，指针，指向构造函数原型对象
console.log(str.constructor.prototype);// String
console.log(str.constructor);// String()
console.log(str.__proto__.constructor);// String()=String.constructor
String.prototype.aa = function() {
	console.log('this is new method');
};
str.aa();
```
1. 姑且认为所有的变量和对象都是直接或间接new某一函数创建的，通常只讨论使用new创建的对象
2. prototype默认为一个对象指针，对象.prototype称该对象的原型/原型对象，该对象包含【所有实例共享的属性和方法】。原型对象的构造函数是一个指针，指回原构造函数
3. 原型对象具有的属性和方法，该对象将自动具有（如constructor属性，指向构造函数本身，`Array.constructor = Array()`）。工具函数一般放置在构造函数上，如`Array.isArray()`
4. 原型对象具有自己的原型对象，如此构成原型链。方法调用过程：自身-原型对象-原型对象.原型对象-...-Object，最终未找到将报错`Object.prototype = null`。
5. 对象可以无条件使用原型对象上挂的属性和方法。不论简单数据类型（值）或引用数据类型（地址），都不能直接修改原型对象（.prototype={}）。但引用数据类型可以更改地址指向的内存空间内容（p1.b.key=11）。
6. 使用new声明的变量，this关键字指向空对象。默认返回该空对象。（首先在内存中创建空对象，之后经过构造函数处理）

## 构造函数和原型
构造函数是一个方法，有它自己的prototype属性，该属性指向构造函数的原型对象。
> js不是一个严格面向对象的语言，主要是没有提供抽象、继承、重载等面向对象机制。但通过原型，可以实现或模拟面向对象的特性，故称基于原型的面向对象。

## Function.prototype
```javascript
// 强行指定被调用函数内部this指向
Function.prototype.call// 如fun.call(10, 10);第一个参数默认this=Number{10}。对已经绑定的this再call将失效。apply语法糖
Function.prototype.apply// 接收两个参数，第一个参数指定函数体内this指向，第二个参数为带下标的参数集合，可以是数组或类数组。返回值为【调用了指定this和参数的函数】的结果

//p1
document.querySelector(".aa").onclick = function(e) {...}
//p2
var handler = document.querySelector(".aa").onclick;
document.querySelector(".aa").onclick = function(e) {
	if(typeof handler === 'function') {
		handler.call(this, e);
	}...
}
-----------------
var fun1 = fun.bind(obj);// 返回一个新的、this指向指定位置的函数
```
## 应用

> 改变this指向；借用其它对象的方法

1. 判断某个变量是否为数组、对象...
```javascript
var a = [];
Object.prototype.toString.call(a) === '[object Array]'// 不能使用a.toString。因为Array存在toString()，原型链机制会导致调用Array.toString。使用call()强制指定Object.prototype.toString()上下文为a，打印目标结果。[object Array]，前半部分总为object，后半部分为对象构造函数名称
Array.isArray(a);// 兼容性稍差

var obj = {};
Object.prorotype.toString.call(obj) === '[object Object]'
```
2. 写一个函数实现字符串反转
```javascript
// reverse()用于颠倒数组中元素的顺序
String.prototype.reverse = function() {
	return this.split('').reverse().join();
};
var str = "";
str.reverse();
```
3. 将类数组`agruments/dom collection等`转化为数组
```javascript
// slice(0)相当copy数组，但类数组不能调用。因slice利用length与[]下标工作，类数组同样具有这个属性。使用call指定this上下文，对返回的新数组进行forEach遍历
function fun() {
	Array.prototype.slice.call(arguments, 0).forEach(function(item) {
		console.log(item);
	});
}
fun(1,2,3,4,5);

Array.from(arguments);//ES6方法，使用范围包括具有length属性的对象
var args = [...arguments];//ES6扩展运算符，使用要求具有便利器接口
var arr = $.makeArray(arguments);//jQuery
```

## this

关键字，指代当前执行上下文

1. 普通定义，作为普通函数直接调用，指向全局对象`window`
2. 普通定义，作为事件处理函数，指向关联dom
3. 普通定义，作为对象{}内方法，指向被调用的对象本身，可以改变指向
4. 普通定义，作为原生js发送`ajax`的回调函数，`ajax`成功后执行时指向`XMLHttpRequest`对象
5. 作为构造函数、使用new运算符调用的函数，默认指向构造函数返回的这个空对象。如果构造器函数中显式的返回了`object`类型的对象，则指向这个显式返回的对象
6. 使用`Function.prototype.call/apply/bind`调用的函数，指向指定的参数对象。使用`bind`绑定的`this`无法再改变
7. 作为`forEach`第一个参数，若`forEach`没有给定第二个参数，指向全局对象。否则指向第二个参数
8. 箭头函数定义，指向声明时作用域`this`，无法改变指向（与`bind`同级）
9. 箭头函数定义，作为`class`内方法，指向调用时的对象，无法改变指向（与普通箭头函数不同的指定规则）
```js
// this作用域（不能被闭包）
var name = 'HUN';
function Hundsun(name) {
    this.name = name;
    this.obj = {
        name: '611',
        foo1: function() {
            return function() {
                console.log(this.name);
            }
        },
        foo2: function() {
            return () => {
                console.log(this.name);
            }
        }
    };
}
var p1 = new Hundsun('hundsun');
var p2 = new Hundsun('aa');

// 浏览器中运行结果：（node中'HUN'改为undefined）

// p1.obj.foo1()为返回的function匿名函数，再次调用为普通函数调用，故this指向全局作用域
p1.obj.foo1()();// 'HUN'
// p1.obj.foo1.call(p2)为foo1指定this为p2，但返回的匿名函数由于是function函数并普通调用，this仍指向全局作用域
p1.obj.foo1.call(p2)();// 'HUN'
// p1.obj.foo1()为返回的function匿名函数，使用call()绑定其中this指向p2
p1.obj.foo1().call(p2);// 'aa'

// p1.obj.foo2()为返回的箭头匿名函数，箭头函数继承父级作用域foo2的this，即foo2中this（指向obj）。
p1.obj.foo2()();// '611'
// p1.obj.foo2.call(p2)为foo2指定this为p2，返回的箭头函数继承foo2的this，指向p2
p1.obj.foo2.call(p2)();// 'aa'
// p1.obj.foo2()为返回的箭头匿名函数，箭头函数忽略call()，this指向foo2中this（指向obj）
p1.obj.foo2().call(p2);// '611'
```

# 防抖 debounce、节流 throttle、分时函数
都是防止某一时间段内被频繁触发
## 防抖
触发事件后，在某一时间段内仅执行一次。如果触发事件后在这段时间内又触发了事件，则重新计算延时时间。
> 本质是延时响应。在事件触发n秒后再执行回调函数。如果n秒内又被触发，则重新计时。【触发后延时n秒执行】
>
> ```javascript
> function debounce(fun, delay) {
>     let timer = null;
>     return function(e) {
>         if(timer !== null) {
>            clearTimeout(timer);
>            timer = null;
>         }
>         timer = setTimeout((function() {
>             fun.call(this, e);
>         }).bind(this), delay);
>     };
> };
> ```
>
> 

应用场景：联想输入

## 节流
以固定的低频率执行代码逻辑。
> 本质是降频。在规定单位时间内，仅触发一次回调函数执行。如果在规定单位时间内某事件被触发多次，只有一次能生效。【触发后立即执行，n秒内不再执行】
>
> ```javascript
> function throttle(fun, delay) {
>     let lock = false;
>     return function(e) {
>         if(lock) return;
>         lock = true;
>         setTimeout(() => lock = false, delay);
>         fun.call(this, e);
>     };
> };
> ```
>
> 

应用场景：鼠标连续点击；页面无限加载场景下，需要用户在滚动页面时，每隔一段时间发一次ajax请求；监听scroll（是否滑到底部、自动加载更多）、resize、mousemove事件等

## 分时函数
在大量数据需要渲染时，为避免js一次执行太多的任务，使用分时函数每个时间段执行一次任务，直到所有任务执行完成
> 【大量任务分多次执行】

```javascript
// 注意每次执行任务的临界值是，count、1、剩余数据中的最小值
// arr 数据，fn 逻辑函数，count 每次完成的任务数
var timeChunk = function(arr, fn, count) {
	var timer = null;
	var start = function() {
		for(var i = 0; i < Math.min(count || 1, arr.length); ++i) {
			fn(arr.shift());
		}
	};
	timer = setInterval(function() {
		if(arr.length === 0) {
			clearInterval(timer);
			timer = null;
		}
		start();
	}, 500);
};
```

# 闭包
在A函数内创建B函数，且B函数使用了A函数的局部变量，则称B函数闭包到了A函数的该局部变量。
闭包在处理速度和内存消耗方面对脚本具有负面影响。在创建新的对象或类MyObject时，方法应关联于对象原型。否则每次创建新对象object，object中的方法都会被重新赋值一次。
> 一个函数和对其周围状态的引用捆绑在一起，这样的组合就是闭包

作用：

1. 让变量成为函数私有变量，（IIFE）减少全局变量污染：debounce中timer为每个input所私有
2. 模拟私有方法
```javascript
var counter =(function() {
	var privateCounter = 0;
	function changeBy(val) { privateCounter += val; }
	return {
		increase: function() { changeBy(1); },
		decrease: function() { changeBy(-1); },
		value: function() { return privateCounter; }
	}
})();
console.log(counter.value());
counter.increase();
```
