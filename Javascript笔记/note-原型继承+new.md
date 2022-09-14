[toc]
# 1.对象基本概念
## 1.1构造函数
Javascript对象体系基于构造函数(constructor)和原型链(prototype)
特点：函数体内部使用this关键字；生成对象时使用new命令

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

## 1.2this
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

## 1.3new

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




## 1.4继承
### 1.4.1原型对象
原型对象的作用，就是定义所有实例对象共享的属性和方法，而实例对象可以视作从原型对象衍生出来的子对象。

### 1.4.2原型链
JavaScript 规定，所有对象都有自己的原型对象（prototype）。一方面，任何一个对象，都可以充当其他对象的原型；另一方面，由于原型对象也是对象，所以它也有自己的原型。如果一层层地上溯，所有对象的原型最终都可以上溯到`Object.prototype=null`

### 1.4.3constructor属性
指向prototype对象所在的构造函数。
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

### 1.4.4instanceof运算符
返回一个布尔值，表示对象是否为某个构造函数的实例
检查整个原型链，因此同一个实例对象可能会对多个构造函数都返回true。当左边对象原型链上只有null对象时，instanceof将判断失真。
应用：判断值的类型（只能用于对象）
```javascript
var x = {};
x instanceof Object//true
```

### 1.4.5构造函数的继承
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

# 2.模块
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

# 3.Object对象相关方法
1. `.getPrototypeOf(arg)`：返回参数对象的原型
2. `.setPrototypeOf(arg1, arg2)`：为参数对象设置原型，返回该参数对象。它接受两个参数，第一个是现有对象，第二个是原型对象
3. `.isPrototypeOf(arg)`：判断该对象是否为参数对象的原型
4. `._proto_`：返回该对象的原型，可读写。只有浏览器才需要部署
5. `.getOwnPropertyNames(arg)`：返回一个数组，成员是参数对象本身的所有属性的键名，不包含继承的属性键名。`Object.keys(arg)`只获取那些可以遍历的属性
6. `.hasOwnProperty(arg)`：判断某个属性是否定义在对象自身（也可能定义在原型链上）

in：返回一个布尔值，表示一个对象是否具有某个属性。不区分是自身的属性还是继承的属性。获得对象的所有可遍历属性（不区分自身与继承）使用for-in循环。
```javascript
for(var name in object){
	if(object.hasOwnProperty(name)){//获得对象自身属性
		//code
	}
}

//对象拷贝
function copyObject(orig){
	return Object.create(
		Object.getPrototypeOf(orig),//确保相同原型
		Object.getOwnPropertyDescriptors(orig)//确保相同实例属性
	);
}
```

