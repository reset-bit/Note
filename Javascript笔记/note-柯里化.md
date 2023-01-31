[toc]

# 函数柯里化--Currying

又称部分求值，只传递给函数一部分参数来调用它，让它返回一个函数去处理剩下的参数。

> 柯里化是把【接收的多个参数的函数】变换成【接收一个单一参数的函数】，并且返回【接收余下的参数、返回结果的新函数】的技术。

简单实现：

```javascript
function curryingAdd(x, y) {
	return function(y) {
		return x + y;
	};
}
curryingAdd(1)(2);// 3
```
## currying优势
1. 参数复用
```javascript
// 正则验证字符串
function curryingCheck(reg) {
	return function(txt) {
		return reg.test(txt);
	};
};
let hasNumber = curryingCheck(/d+/g);
let hasLetter = curryingCheck(/[a-z]+/g);
hasNumber('test1');// true
hasLetter('111');// false
```
2. 延迟执行：没有立即执行，只有当再次调用的时候再执行
```javascript
Function.prototype.bind = function(context) {// context为希望修正的this对象
	var _this = this;// 暂存原先函数
	var args = Array.prototype.slice.call(arguments, 1);// 截取第二个到第最后一个实参，即所有参数
	return function() {
         var _args = [...arguments];
		return _this.apply(context, args.concat(_args));// 应用指定上下文
	};
};
// -------------------
Function.prototype._bind = function(context) {
    var _this = this;// fn
    var args = Array.prototype.slice.call(arguments, 1);// ['z']
    return function() {
        var _args = Array.prototype.slice.call(arguments);// [0]
        return _this.apply(context, args.concat(_args));
    };
};

x = 1;
let obj = { x: 2 };
let t = function(a) {
    console.log(a);// z, 0未接收
    console.log(this.x);// 2
};
t._bind(obj, 'z')(0);// z[\n] 2
```

## 通用封装方法
```javascript
// 初步封装，仅支持currying(a)(b);
var currying = function(fn) {
	var args = Array.prototype.slice.call(arguments, 1);// 获取第一个方法内全部参数，截取的是currying-argument从第二个开始的元素，第一个元素是原上下文fn
	return function() {
		var newArgs = args.concat([...arguments]);// 将后面方法里的全部参数和args进行合并
		return fn.apply(this, newArgs);
	};
};
```

## 性能
- 存取【argument对象】要比存取【命名参数】慢
- 使用fn.apply()和fn.call()比直接调用fn()慢
- 创建大量嵌套作用域和闭包函数会带来内存和速度上的花销
> 大部分应用中，主要性能瓶颈在操作dom节点上，currying性能损耗基本可以忽略

## 相关题目
```javascript
// 实现一个add方法，使计算结果能够满足如下预期：
add(1)(2)(3) == 6;
add(1, 2, 3)(4) == 10;
add(1)(2)(3)(4)(5) == 15;

// 柯里化：参数复用。函数自身仅作参数合并功能
function add() {
    // 第一次执行时，定义一个数组专门用来存储所有的参数
    var _args = Array.prototype.slice.call(arguments);

    // 在内部声明一个函数，利用闭包的特性保存_args并收集所有的参数值
    var _adder = function() {
        _args.push(...arguments);
        return _adder;
    };

    // 由于typeof add(1, 2) === 'function'，非'number'/'string'，所以在'=='时，调用toString方法，隐晦的完成功能
    _adder.toString = function () {
        return _args.reduce(function (a, b) {
            return a + b;
        });
    }
    return _adder;
}

add(1)(2)(3) == 6// true
add(1, 2, 3)(4) == 10// true
add(1)(2)(3)(4)(5) == 15// true
add(2, 6)(1) == 9// true
```

# uncurrying
泛化函数this。相比于curry，没有捕获回调函数参数，而是将回调函数this绑定为首参。
```javascript
Function.prototype.uncurrying = function() {
	let self = this;
	return function() {
		let obj = Array.prototype.shift.call(arguments);// arguments中首元素
		return self.apply(obj, arguments);
	};
};
var push = Array.prototype.push.uncurrying();
var obj = { 'length': 1, '0': 1 };
push(obj, 2);// {'0': 1, '1': 2, 'length': 2}
```