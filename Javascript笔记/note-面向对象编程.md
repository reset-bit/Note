[toc]
# 面向对象编程

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

## 1.1 constructor
指向prototype对象所在的构造函数。构造函数是一个方法，有它自己的prototype属性，该属性指向构造函数的原型对象。

> js不是一个严格面向对象的语言，主要是没有提供抽象、继承、重载等面向对象机制。但通过原型，可以实现或模拟面向对象的特性，故称基于原型的面向对象。

```javascript
var Vehicle = function(){
	'use strict';//严格模式，this不能指向全局对象。未使用new命令时将报错
	this.price = 1000;
	return 100;//new将忽略return语句，返回构造后的this对象
	//return {price:200};new将返回这个新对象
}
//等价
var v = new Vehicle();//若对普通函数使用new，则返回一个空对象
var v1 = Object.create(v);//以现有对象为模板，生成新的实例对象
var v2 = Vehicle.create(Vehicle.prototype);
```

构造函数与原型对象：构造函数通过其属性prototype去寻找它关联的原型，如果用M表示构造函数，M.prototype所指的就是它关联的原型对象。而原型对象可以通过构造器constructor来寻找与自身关联的构造函数，所以就有M.prototype.conctructor===M。【构造函数是关联原型对象里的一个函数。】如果修改了原型对象，一般会同时修改constructor属性，防止引用的时候出错。
应用：获取实例对象的构造函数；从一个实例对象新建另一个实例

```javascript
function Constr() {}
var x = new Constr();

x.constructor == Constr//true
x.constructor.name //Constr
var y = new x.constructor();

C.prototype.method = function(){...};//在原型对象上添加方法，保证instanceof不失真
```

## 1.2 this
总是返回属性或方法当前所在的对象（换句话说，仅支持保存一层环境）
实质：引擎在内存中生成对象，之后将对象地址赋值给变量。但当对象中含有值为函数的属性时，这时引擎将函数单独保存在内存中，之后将函数地址赋值给对象属性。由于该函数也可以单独执行，故this出现。并在函数体内部指代函数当前的运行环境。

变量固定this值：
```javascript
var o = {//地址1
  f1: function () {//地址2
    console.log(this);
    var f2 = function () {//IIFE，直接执行地址3，运行环境为全局变量
      console.log(this);
    }();
  }
}
o.f1();//从地址1调用地址2，指向o；
//{ f1: [Function: f1] }
//Object [global]{...} ==> window

//相当于：
var temp = function () {
  console.log(this);
};
var o = {
  f1: function () {
    console.log(this);
    var f2 = temp();
  }
}

//解决：
var o = {
  f1: function() {
    console.log(this);
    var that = this;
    var f2 = function() {
      console.log(that);
    }();
  }
}
```

绑定this：
1. `function.prototype.call(obj(,args))`：改变function作用域为obj，agrs为函数调用所需要的参数。应用：调用对象的原生方法，无论继承后的方法有没有被覆盖，都不影响：`Object.prototype.function.call(obj, arg);`
2. `function.prototype.apply(obj(,[arg1...]))`：同上一函数，仅参数形式不同。应用：找出数组最大元素：`Math.max.apply(null, [10, 2, 15]);//15`；将数组空元素变为undefined（遍历时显示undefined）：`Array.apply(null, ['a',,'b']);//['a',undefined,'b']`
3. `function.prototype.bind(obj(,args))`：将函数体内的this绑定到某个对象，然后返回一个新函数。相当于携带环境调用函数。支持接收多个参数，后继的参数将绑定原函数的参数，因此再次调用时可减少参数数量。注意添加监听事件的时候bind返回匿名函数，不能直接写在监听事件函数内。
```javascript
var add = function(x, y){
	return x * this.m + y * this.n;
}
var obj = {m:2,n:2};
var newAdd = add.bind(obj, 5);
newAdd(5);//20
```

常见指向：

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

## 1.3 new

```javascript
function myNew() {
    // 以构造函数为原型创建源对象 ==> 创建新对象->获取构造函数->连接原型
    let obj = new Object();
    var constructor = Array.prototype.shift.call(arguments);
    obj.__proto__ = constructor.prototype;
    // 将this、初始化参数传递给构造器
    let ret = constructor.apply(obj, arguments);
    // 如果构造函数没有返回显式对象，则返回第一步创建的新对象
    return typeof ret === 'object' ? ret : obj;
};
```


## 1.4 继承

### 1.4.4 instanceof运算符
返回一个布尔值，表示对象是否为某个构造函数的实例
检查整个原型链，因此同一个实例对象可能会对多个构造函数都返回true。当左边对象原型链上只有null对象时，instanceof将判断失真。
应用：判断值的类型（只能用于对象）
```javascript
var x = {};
x instanceof Object//true
```

### 1.4.5 继承方式
```javascript
function Shape(){this.x=0;}
Shape.prototype.move = function(dis){
	this.x += dis;
}
function MyObject(){this.category='object';}
function Rectangle(){
	Shape.call(this);//调用父类构造函数，将具有父类实例的属性
}
//子类继承父类的原型（整个继承）
Rectangle.prototype = Object.create(Shape.prototype);//使得Rectangle原型对象为Shape，并挂在Shape原型链上
Rectangle.prototype.constructor = Rectangle;//保证子类构造函数指向正确，否则指向Shape
//多重继承（同时继承多个对象，Mixin）
Object.assign(Rectangle.prototype, MyObject.prototype);//继承链上加MyObject
//单个方法继承
Rectangle.prototype.move = function(){
	Shape.prototype.move.call(this);
	//some code
}

var rec = new Rectangle();
console.log(rec.prototype)//undefined，需另定义
console.log(Shape.prototype)//匿名原型对象，但含有move方法
console.log(Rectangle.prototype)//Shape
```

#### 1.4.5.1 `prototype`原型链继承

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

#### 1.4.5.2 构造函数

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

#### 1.4.5.3 组合继承（伪经典继承）

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

#### 1.4.5.4 原型式继承

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

#### 1.4.5.5 寄生继承

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

#### 1.4.5.6 寄生组合继承

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

#### 1.4.5.7 ES6

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

