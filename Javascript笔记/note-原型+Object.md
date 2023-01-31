[toc]

# 原型与原型链

在 JavaScript 的语言特性中没有“类”的概念，为了便于理解，用【类】来称呼那些实际上将会调用构造函数的 Function 对象，用【对象实例】来称呼并强调通过调用该对象的构造函数而生成的对象。

某个【对象】的原型/原型对象就是所有该【对象实例】共享的一组属性和方法。这些属性和方法定义在这些对象实例的构造函数`prototype`属性中。`prototype`是一个指针，指向对象的原型对象。也可以通过实例对象的`__proto__(dunder proto)`属性获取构造函数原型对象。原型对象本身也可能有原型，并从中继承属性和方法，以此类推。这种关系被称为原型链。调用方法时，js引擎首先在当前对象上寻找是否具有该方法，有就调用，没有就将该请求委托给原型对象。如果在原型对象上也没有该方法，就委托给到原型对象的原型对象，以此类推。js中，根对象是`Object.prototype`，它的原型为null。**未指定原型对象的`function`原型对象为{}。**

> 关于\__proto\__和prototype：
>
> 1. 实例化对象有属性\__proto\__，指向该对象的构造函数的原型对象；
> 2. 方法除了有属性\__proto\__，还有属性prototype，prototype指向该方法的原型对象

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

**使用new声明的对象创建过程：**

1. 以构造器的`prototype`属性为原型创建新对象
2. 将创建新对象的this和构造函数需要的参数传给构造器，令其执行
3. 如果构造器没有返回对象，则返回第一步创建的对象
```javascript
function myNew() {
    let obj = new Object();
    let constructor = Array.prototype.shift.call(arguments);
    obj.__proto__ = constructor.prototype;// 关联原型对象
    let ret = constructor.apply(obj, arguments);// 传递this和参数
    return typeof ret === 'object' ? ret : obj;
};
```

PS：

1. 姑且认为所有的变量和对象都是直接或间接new某一函数创建的，通常只讨论使用new创建的对象
2. prototype默认为一个对象指针，对象.prototype称该对象的原型/原型对象，该对象包含【所有实例共享的属性和方法】。原型对象的构造函数是一个指针，指回原构造函数
3. 原型对象具有的属性和方法，该对象将自动具有（如constructor属性，指向构造函数本身，`Array.constructor = Array()`）。工具函数一般放置在构造函数上，如`Array.isArray()`
4. 原型对象具有自己的原型对象，如此构成原型链。方法调用过程：自身-原型对象-原型对象.原型对象-...-Object，最终未找到将报错`Object.prototype = null`。
5. 对象可以无条件使用原型对象上挂的属性和方法。不论简单数据类型（值）或引用数据类型（地址），都不能直接修改原型对象（.prototype={}）。但引用数据类型可以更改地址指向的内存空间内容（p1.b.key=11）。
6. 使用new声明的变量，this关键字指向空对象。默认返回该空对象。（首先在内存中创建空对象，之后经过构造函数处理）

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



# Object.xx

1. `.getPrototypeOf(arg)`：返回参数对象的原型
2. `.setPrototypeOf(arg1, arg2)`：为参数对象设置原型，返回该参数对象。它接受两个参数，第一个是现有对象，第二个是原型对象
3. `.isPrototypeOf(arg)`：判断该对象是否为参数对象的原型
4. `._proto_`：返回该对象的原型，可读写。只有浏览器才需要部署
5. `.getOwnPropertyNames(arg)`：返回一个数组，成员是参数对象本身的所有属性的键名，不包含继承的属性键名。`Object.keys(arg)`只获取那些可以遍历的属性
6. `.hasOwnProperty(arg)`：判断某个属性是否定义在对象自身（也可能定义在原型链上）
7. `.defineProperty(arg)`：直接在对象上定义一个新属性，或者修改已有属性。返回值为新对象
	
	```js
	Object.defineProperty(obj, propertyName, {
	    value: '',
	    writable: false | true,// 是否可修改，严格模式下再修改
	    enumerable: false | true,
	    configurable: false | true,// 是否可配置，配置为false之后再配置会报错
	    set: function() {},
	    get: function() {}
	});
	```

8.`.is(val1, val2)`：对+0与-0、NaN与NaN会返回正确结果。并不是比===和==严格，只是有更特殊的用途

9.`.assign(target, source)`：伪深拷贝，另建地址拷贝，仅拷贝一层。returnTarget === target，合并source到target并返回

10.`.create(obj)`：返回一个新对象，令该对象的__proto\_\_为obj
