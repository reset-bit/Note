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

# 面向对象编程

## 类与实例对象

第一阶段：使用`function`作为构造函数，对象内方法各自持有，浪费内存

第二阶段（ES5）：引入原型概念，所有实例对象共享原型对象上的方法，对公共的只读属性同样适用于挂载到原型对象上。

```javascript
function Person() {};
Person.prototype.name = '佚名';
Person.prototype.detail = [160, 80];
var zhangsan = new Person();
zhangsan.name = 'zhangsan';
zhangsan.detail[1] = 50;
var lisi = new Person();
lisi.name = 'lisi';

console.log(Person.prototype.name);// '佚名'
// 当属性为基本数据类型时，实例对象修改属性值不会引起原型对象的属性值改变。
console.log(zhangsan.name);// 'zhangsan'
// 当属性为引用数据类型时，
consoel.log(lisi.detail);// [160, 50]，引用数据类型，改变内存内容
```

第三阶段（ES5）：构造函数+原型

```javascript
// 常常在构造函数中定义属性，使用prototype属性将方法分装在不同代码块中，使代码更具有可读性
var Student = (function() {
    var _a = '';// 静态公共私有变量
    function StudentConstructor() {
        var _b = '';// 私有变量，对外不可调用
        this.className = '1年1班';
    };
    StudentConstructor.prototype.getInfo = function() {
        console.log(this.className + " " + this.name);
    };
    return StudentConstructor;
})();
console.log(Student.__proto__);// {}
var zhangsan = new Student();
zhangsan.name = 'zhangsan';
zhangsan.getInfo();// "1年1班 zhangsan"
var lisi = new Student();
lisi.name = 'lisi';
lisi.getInfo();// "1年1班 lisi"
```

第四阶段（ES6）：使用class关键字，语法更加简洁

```javascript
// 利用IIFE产生的闭包，封装私有变量。缺：写法复杂，有额外性能开销
const Person = (function() {
    static let _a = '';
    let _name;// 私有变量
    class Person {// 不存在变量提升
        constructor(name) {
            _name = name;
        }
        showName() {
            return _name;
        }
    }
    return Person;
})();
let zhangsan = new Person('zhangsan');
console.log(zhangsan.showName());// 'zhangsan'
```

## 继承

### 1.`prototype`原型链继承

```javascript
function Person() {};
Person.prototype.say = function() {};
function Teacher() {};
Teacher.prototype = new Person();// Teacher.prototype.prototype = Person.prototype
Teacher.prototype.constructor = Teacher;// 修改构造函数
Teacher.prototype.teach = function() {};
// 优：简单易用；子类对象是子类的实例，也是父类的实例；父类新增的属性和方法，子类都能访问
// 缺：为子类添加方法只能在new Person()之后；所有子类共享父类属性

// 在父类构造函数中没有动态开辟的属性时建议采用
```

### 2.构造函数

```javascript
function Person(name) {};
Person.prototype.say = function() {};
function Teacher(name, major) {
    // this: {}
    Person.call(this, name);// this: {name: name}
};
Teacher.prototype.teach = function() {};
// 优：子类对象拥有父类构造函数内的成员变量和方法
// 缺：子类对象无法拥有父类prototype上的属性和方法；子类对象只是子类的实例，不是父类的实例
```

### 3.组合继承（伪经典继承）

```javascript
function Person(name) {};
Person.prototype.say = function() {};
function Teacher(name, major) {
    Person.call(this, name);// 一份
};
Teacher.prototype = new Person();// 一份
Teacher.prototype.constructor = Teacher;
Teacher.prototype.teach = function() {};
// 缺：子类对象上具有的父类属性方法，在原型对象上还具有一份，浪费内存。表现形式为：删除delete t.name，仍然能够访问到name属性，只是是在原型对象上的属性
```

### 4.原型式继承

```javascript
function Person(name) {};
Person.prototype.say = function() {};
var p = new Person('zhangsan');
var t = Object.create(p);// 返回一个【新对象】，使用参数对象来作为新创建对象的__proto__
// 即，t.__proto__ ~= p;
/**
	function create(o) {
		F = function() {};
		F.prototype = o;// 一份
		return new F();// 一份
	};
*/
// 缺：子类对象上具有的属性和方法，在子类的原型对象上还有一份

// 适用于：构建未知构造函数的实例对象
```

### 5.寄生继承

```js
function Person(name) {};
Person.prototype.say = function() {};
var p = new Person('zhangsan');
var t = create(p);
function object(o) {
    F = function() {};
    F.prototype = o;
    return new F();
};
function create(ori) {
    var clone = object(ori);
    clone.teach = function() {};
    return clone;
};
// 创建一个用于封装继承的函数，该函数在内部以某种方式增强对象
```



### 6.寄生组合继承

```javascript
function Person(name) {};
Person.prototype.say = function() {};
function Teacher(name, major) {
    Person.call(this, name);
};
Teacher.prototype = Object.create(Person.prototype);// 相较于组合继承的唯一区别，以Person.prototype为实例对象创建该对象的实例对象（寄生）。没有了Person构造函数的私有变量，在Teacher构造函数中使用Person.call()已经调用。
Teacher.prototype.constructor = Teacher;
Teacher.prototype.teach = function() {};
// 建议采用
```

### 7.ES6

```javascript
class Person {
    constructor(name) { this.name = name; }
    say() {}
}
class Teacher extends Person {
    constructor(name, major) {
        super(name);
        this.major = major;
    }
    teach() {}
}

// 限定this
class Person {
    constructor(name) {}
    say = () => {};// 箭头函数限定总指向调用时的对象本身
}
class Teacher extends Person {
    constructor() {}
    teach = () => {};
}
let t = new Teacher('zhangsan');
t.say.call({name:'lisi'});// 'zhangsan'
```

