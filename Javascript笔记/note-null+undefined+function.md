[toc]
# null && undefined
相同点：if语句中转false；null == undefined
不同点：null表示空的对象，转为数值时为0；undefined表示此处无定义的原始值，转为数值时为NaN

返回undefined：
1. 变量声明但未赋值
2. 对象未赋值的属性；数组未初始化的元素或越界访问
3. 调用函数时未提供参数，该参数为undefined
4. 函数没有返回值时，返回undefined



# 函数

函数只是一个可以执行的值。可以把函数赋值给变量和对象的属性、当作参数传入其它函数、作为函数的结果返回。

## 声明方法
```javascript
//function命令，存在变量提升
function print(){}

//函数表达式，不存在变量提升
var print = function(){};//将匿名函数赋值给变量
var print = function x(){};//该函数名仅在函数体内部有效
var print = new function() {};//这是一个匿名的用户自定义【对象】，在大括号中会构建一个变量作用域，this将指代这个作用域本身
var print = new Function('...');//使用系统内置函数对象来构建一个函数，函数体以字符串形式给出
var print = Function('...');//效果同上
```
若同一个函数被多次声明，则后面的声明将覆盖前面的声明
> 不论全局作用域或是函数作用域内部都存在变量提升现象：var声明的变量不管在什么位置，变量声明都会被提升到函数体的头部
> javascript引擎将函数名视同变量名，采用function命令声明函数时，整个函数将被提升到代码头部。
> 若采用function命令和var赋值语句声明同一个函数，最后会采用var赋值语句的定义。若采用function命令和var声明同一个变量，最后会采用function语句的定义。

## 函数属性及作用域
`.name`属性：获取参数函数的名字
`.length`属性：返回函数预期传入的参数个数
`.toString`属性：返回函数源码字符串，包括函数名及注释

函数作用域是声明时所在的作用域，在函数作用域内声明的变量在函数外无法访问。

## 函数参数
1. 函数运行时不论提供几个参数都不会报错，省略参数的值就变为undefined
2. 若函数参数是复合类型的值（数组、对象、其他函数），传递方式是传址传递。即，在函数内部修改参数，将会影响到原始值。但在函数内部替换掉整个参数不会影响到原始值。（重新赋值相当指向另一个地址，此时原地址内容没有发生改变，参数仍为原来的值，因为不能改变传入的参数地址）若函数参数是原始类型的值（数值、字符串、布尔值），传递方式是传值传递。
3. 同名参数取最后出现的那个值
4. 调用函数时令形参赋值，该值是形参默认值

### arguments对象
类数组对象，包含函数运行时的所有参数，访问如`arguments[0]`，仅在函数体内部可以使用。总是与实际传递的实参相关联。形参个数多于实参，则形参为`undefined`。在非严格模式（且不存在默认参数）下，`arguments`和形参变量存在映射机制；在严格模式下，二者不存在映射机制。

常用应用场景：未知参数的参数调用	

相关属性：
`.length`属性：用于判断函数调用时携带参数个数；
`.callee`属性：返回对应的原函数（严格模式禁用）

```js
function side(arr) {arr[0] = arr[2];}
function a(a, b, c = 3) {
    c = 10;// 改变的是形参c，不是arguments[2]
    side(arguments);
    console.log(arguments);// [1,1,1]
    return a + b + c;// 1 + 1 + 10
}
console.log(a(1, 1, 1));// 12
// ---------------
function a(a, b, c) {
    c = 10;// 改变的是形参c和arguments[2]
    side(arguments);
    console.log(arguments);// [10,1,10]
    return a + b + c;// 10 + 1 + 10
}
console.log(a(1, 1, 1));// 21
```

## 立即调用的函数表达式IIFE
通常只对匿名函数使用IIFE，避免污染全局变量且利于封装外部无法读取的私有变量。
```javascript
var i = function(){return 10;}();
(function(){return 10;}());
```

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

