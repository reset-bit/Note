[toc]

# 防抖 debounce、节流 throttle、分时函数

都是防止某一时间段内被频繁触发

## 防抖

触发事件后，在某一时间段内仅执行一次。如果触发事件后在这段时间内又触发了事件，则重新计算延时时间。

> 本质是延时响应。在事件触发n秒后再执行回调函数。如果n秒内又被触发，则重新计时。【触发后延时n秒执行】
>
> ```
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

应用场景：联想输入

## 节流

以固定的低频率执行代码逻辑。

> 本质是降频。在规定单位时间内，仅触发一次回调函数执行。如果在规定单位时间内某事件被触发多次，只有一次能生效。【触发后立即执行，n秒内不再执行】
>
> ```
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

应用场景：鼠标连续点击；页面无限加载场景下，需要用户在滚动页面时，每隔一段时间发一次ajax请求；监听scroll（是否滑到底部、自动加载更多）、resize、mousemove事件等

## 分时函数

在大量数据需要渲染时，为避免js一次执行太多的任务，使用分时函数每个时间段执行一次任务，直到所有任务执行完成

> 【大量任务分多次执行】

```
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

在A函数内创建B函数，且B函数使用了A函数的局部变量，则称B函数闭包到了A函数的该局部变量。 闭包在处理速度和内存消耗方面对脚本具有负面影响。在创建新的对象或类MyObject时，方法应关联于对象原型。否则每次创建新对象object，object中的方法都会被重新赋值一次。

> 一个函数和对其周围状态的引用捆绑在一起，这样的组合就是闭包

作用：

1. 让变量成为函数私有变量，（IIFE）减少全局变量污染：debounce中timer为每个input所私有
2. 模拟私有方法

```
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