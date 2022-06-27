[toc]
# Javascript
## Dom
### window.load()与body.onload
使用`window.load()`而不用`body.onload=''`的原因：
- onload同时加载多个函数时，必须`onload="f1(), f2()"`。load()可以写成`window.load(f1() {}); window.load(f2() {});`
- onload不能做到js和html完全分离
	
	> 二者都在页面所有元素（包括HTML标签、引用得到的所有图片等）加载完毕后执行。同时`$(docuemnt).ready(function() {})/$(function() {})`是文档结构已经加载完成，不必等到所有加载完毕。同时支持多次绑定事件。

### 音/视频
- `currentTime`设置或返回视频播放的当前位置（s），设置该属性，使其跳转到指定位置。
- `duration`设置动画过渡时间。
- `defaultPlaybackRate`设置或返回音/视频播放的默认速度
- `playbackRate`设置或返回音/视频当前的返回速度

### innnerHTML & innerText

```html
<html>
	<head><title>innerHTML</title></head>
	<body>
		<div id="d1"><p id="p1">hello world </p></div>
		<script>
			var content = document.getElementById("d1");
			alert(content.innerHTML);// <p id="p1">hello world </p>
			alert(content.innerText);// hello world
		</script>
	</body>
</html>
```



## 变量
1. 未声明直接赋值不报错；未声明直接使用报错`Referrence error`；声明直接使用值为undefined
2. 只要undefined参与运算，结果总为NaN

## 函数
1. 将多行js代码用大括号包起来，整体命名，以便反复调用
2. 注意函数绑定时间与函数调用时间
3. 实参与形参个数无绝对关系
4. `function Person() {}; let obj1 = new Person(); let obj2 = Person();`则`obj1`中`this`指向`obj1`，`obj2`中`this`指向`window`




## 事件
无时无刻都在发生，仅仅是否指定处理事件。
```javascript
onmouseover/onmouseout// 鼠标经过自身及其子元素时触发事件
onmouseenter/onmouseleave// 鼠标经过自身触发事件，经过子元素不触发事件
```
### 事件对象event
```javascript
e.target// 标记指向元素
e.fromElement// 标记从哪个元素移入的
e.delegateTarget// 标记绑定事件对象本身
```

