[toc]

ES6新特性是指，2015.10以后，js出现的新特性

# 关键字

ES6在var和function基础上，添加了4种声明变量的方法：分别是let/const/import/class

## let
1. let声明的变量，仅在let命令所在的代码块内有效；var在全局范围内有效
2. let不存在变量提升，变量需要先声明后使用
3. 暂时性死区TDZ：在代码块内，使用let声明变量之前，该变量都是不可用的，属于死区。在死区使用变量将报错，主要是为了减少运行时错误
4. 不允许重复声明变量，包括let与let、let与var

> let实际上为js在全局作用域、函数作用域（var）之上，新增了块级作用域，避免了计数变量溢出为全局变量等情况。块级作用域允许任意嵌套，且外层作用域无法读取内层作用域的变量。实际上使IIFE不再必要（使用**{}**代替(function() {})();）
>
> 函数可以在块级作用域之中声明，声明语句类似let。为了兼容老代码，ES6规定ES6浏览器的实现可以有自己的行为方式。即【允许在块级作用域内声明函数，函数声明将会提升到全局作用域或函数作用域的头部，同时提升到所在块级作用域的头部】。因此应当避免在块级作用域内声明函数。如果确实需要，应当写成函数表达式，即`let f = function() {}`
>
> ES5只有**全局作用域、函数作用域**。ES6具有全局作用域、函数作用域、块级作用域。
>
> **变量提升**针对当前作用域而言。
>
> ES5：不在任何函数内定义的变量就具有全局作用域，在函数体内部声明的变量就具有函数作用域。**函数作用域中的变量可能会覆盖全局作用域中的变量**。IIFE实际上是为了作用域隔离，因为只有function能实现作用域隔离。
>
> ES6：在一个{}声明的变量具有块级作用域。**外层作用域无法读取内层作用域的变量，内层作用域可以覆盖外层作用域的同名变量**。
>
> ```javascript
> function foo() {
>        var a = 1;
>        console.log(a);// 1
> }
> foo();
> console.log(a);// throw ReferenceError引用不存在变量错误
> ```
>
> 


## const
1. 声明一个只读的常量。一旦声明，就必须立即初始化。
2. const作用域与let相同，块级作用域+变量不提升+暂时性死区+不可重复声明
3. const声明复合类型的变量，仅保证指向数据所在的地址不变。冻结对象可使用`Object.freeze()`方法
	
	> `Object.freeze()`返回被冻结对象。被冻结的对象不能添加、更改属性值。常规模式下更改不报错但无法修改值。冻结的是堆内存中的值，与栈无关。可以增加性能

> JavaScript变量存储机制：对于原始类型，变量存储在栈内；对于复合类型，栈内存储堆内地址的引用。【堆】是一个很大的存储控件，支持存放任何类型的数据。操作系统本身不会清理其内容，需要程序员手动清除，防止内存泄漏。在高级程序设计语言如Java中，会有垃圾回收机制，自动清理堆内存；【栈】是内存中一块用于存储局部变量和函数参数的一块线性结构，先进后出。操作系统自动回收栈内存，栈中变量在函数调用后就会消失
> 
> ES5中，浏览器中【顶层对象】是window，Node中顶层对象是global。ES6规定，var/function声明的变量仍是全局对象的属性，但let/const/class声明的变量，不属于顶层对象的属性
> 

声明常量必须特别注意：

```js
const a = new String('some');
a.foo = 1;
console.log(a);// [String: 'some'] {foo:1}
console.log(typeof a);// object

const a = String('some');
a.foo = 1;
console.log(a);// 'some'
console.log(typeof a);// string，万物皆对象，本身就有length属性

const a = 'some';
a.foo = 1;
// a = 'any';// TypeError
console.log(a);// 'some'
console.log(typeof a);// string
```

## static

定义静态方法和属性：

1. 隶属于当前类，而不属于类的实例，只能通过类名调用
2. 不能被子类继承
3. 静态方法调用同一个类的其他静态方法，可使用this关键字



# 箭头函数

更适用于本来需要匿名函数的地方，不能用作构造函数与方法函数，没有自己的`this/arguments`，没有`prototype属性`
> 不兼容ie

## 基础语法
```javascript
// 代码体内仅有一行代码时，{return;}要省全省
(param1, param2, ..., paramN) => { statements }
(...) => expression
singleParam => expression
singleParam => ({ foo: bar })
(param = defaultValue) => { ... }
// 获取自身参数，使用剩余运算符
let obj = {};
obj.fn = function() {
    let arrow = (...args) => {
        console.log(arguments);// 1,2,3
        console.log(args);// 4,5,6
    };
    arrow(4, 5, 6);
};
obj.fn(1,2,3);
```

## 注意事项
1. 箭头函数从自己的作用域链的上一次继承this
2. 对箭头函数调用`call()/apply()/bind()`会忽略this



# 类

在ES6中，class (类)作为对象的模板被引入，可以通过 class 关键字定义类。

class 的本质是 function，可以看作一个语法糖，让对象原型的写法更加清晰、更像面向对象编程的语法。方法本质还是定义在prototype上。

```js
// 匿名类
let Example = class {
    constructor(a) {// 创建类的实例化对象时调用
        this.a = a;
    }
}
// 命名类
let Example = class Example {
    constructor(a) {
        this.a = a;
    }
}
// 类声明
// 不可重复声明，声明不可提升
class Example {
    constructor(a) {
        this.a = a;
    }
}

class Example = {
    static a = 2;// 静态属性：直接定义在类中的属性，无需实例化
    static sum(a, b) { return a + b }// 静态方法

	a = 2;
    constructor() {
        console.log(this.a);// 实例属性：生成实例后访问，不同的
        this.sum = (a, b) => { return a + b; }// 实例方法
    }

	sum(a, b) { return a + b; }// 原型方法：生成实例后访问，相同的访问地址
}
Example.a = 2;// 静态属性
Example.sum = (a, b) => a + b;// 静态方法
Example.prototype.a = 2;// 原型属性
Example.prototype.sum = (a, b) => a + b;// 原型方法
```



# weakMap & weakSet

Map：key-value，key唯一；Set：元素唯一

相比于Map和Set，weakMap和weakSet持有的是每个对象（键）的弱引用，这意味着即使有其他引用存在时，垃圾回收机制也能正确运行。实际应用中，常将dom对象放入weakMap/weakSet中，起到dom缓存的作用

正是由于弱引用，weakMap和weakSet不可遍历

```js
// 内存泄漏
const obj = {};
const map = new Map();
map.set('o1', obj);
obj = null;

// weakMap-key为对象
const wm = new WeakMap();
const o1 = { name: 'foo' };
wm.set(o1， 1);
wm.has(o1);// true
wm.get(o1);// 1
wm.delete(o1);
// weakSet
const ws = new WeakSet();
ws.add(o1);
ws.has(o1);// true
ws.delete(o1);
```

weakMap的另外一种用途是保留私有数据

```js
const wm = new WeakMap();
class Counter {
    constructor() {
        wm.set(this, {
            _count: 0
        });
    }
    getCount() {
        return wm.get(this)._count;
    }
}
```



# ...扩展收缩运算符

```javascript
// 读，扩展开数组成为多个项（在等号右边）
var arr = [1,2,3];
console.log(...arr);

// 写，收集多个项成为一个数组。仅支持写在最后。（在等号左边）
function test(...args) {
    console.log(args);
};
test(1,2,3);

// 应用：
// 1.数组合并，扩展
var arr1 = [1,2,3], arr2 = [4,5,6];
console.log([...arr1, ...arr2]);// [1,2,3,4,5,6]
// 2.多数字求和
function getSum(...args) {
    return args.reduce((pre, cur) => {// pre是上次调用返回的值，故+=
        return pre += cur;
    }, 0);
};
console.log(getSum(1,2,3));// 6
```



# 简写相关（不仅包括ES6）

```javascript
// 对象键值对简写
var obj = { a: function() {} };
var obj = { a() {} };
var name = 'zhangsan'; var person = { name: name };
var name = 'zhangsan'; var person = { name };

// 解构赋值
var obj = { a: 100, key: 'zhangsan', abc: { aa: 1 } };
var { a:b, key, abc: {a}, c=a } = obj;// 按键名解构，解构3个变量，不包括abc；将a赋别名为b；令c=a

// 函数默认值，需要写在不带默认值参数的后面
function (a, b = 1) {};

// 模板字符串
const db = `http://${host}:${port}/${database}`;

// ...
const odd = [1, 3, 5];
const nums = [2, 4, 6, ...odd];
const [first, ...rest] = [1, 2, 3, 4, 5];// first:1; rest:[2,3,4,5]
const arr = [...'hello'];// ['h','e','l','l','o'] 将字符串转化为真正的数组（任何Iterator接口的对象，如document.querySelectorAll()返回值、map、set等）


// 非ES6
// ==>
// 位运算
// 向下取整
console.log(0.8 | 0);// 0
console.log(0.8 >> 0);// 0，右移
console.log(~~0.8);// 0，双非not
// attention：不可对负数取整，仅由于32位->舍小数位->转变为负数
console.log(-0.8 | 0);// 0，实为-1
// 向上取整
console.log(1.5 + 0.5 | 0);// 2
// attention: 负数只能舍，不能入
console.log(-0.8 + 0.5 | 0);// 0

// undefined
void 0;// 快
0[0];

// Infinity
1/0;

// boolean
!0;// true
!1;// false

// 十进制指数
10000000000 === 1e10
```

