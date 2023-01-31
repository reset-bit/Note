[toc]

# 变量

1. 大小写敏感，仅骆驼命名法（cuiDongPing）+Pascal命名法（Cuidongping）

2. 单双引号可相互替换，需要前后统一，配对出现

3. 变量使用骆驼命名法，建议声明时赋值；允许重复声明+随意切换类型
	
	> 不能在全局声明变量为name
	
4. 变量提升
	> 片面的来说，javascript是解释型语言。解释型语言在执行时，利用解释器将编程语言转换为机器语言来执行，整个过程中没有中间代码。另一方面，几乎在执行后一瞬间就开始，但是没有任何代码优化，所以每一条语句都是分开转换的。
	>
	> 针对变量提升(hoisting)现象，其执行过程为：V8引擎进入执行具体代码的执行上下文（函数），对代码进行词法分析或分词。分析完整个作用域后，将token（foo = 10）解析为抽象语法树（AST）。【在分词时】每次遇到声明语句，就传递该声明到作用域（scope）创建绑定，为其分配内存。之后每次遇到赋值或取值，引擎将通过scope查找绑定，生成CPU可以执行的机器码。==》变量提升只不过是执行上下文的小把戏
	>
	> 关于即时编译（JIT），JIT不是完整的编译器，在执行前进行编译。
	> 详见：https://blog.csdn.net/qq_38836118/article/details/98878286



# dom

window代表整个浏览器（包括地址栏、前进……），可简写省略。声明的全局变量相当为window动态添加属性。
```javascript
	window.location.href = "";
	window.alert("");
	window.confirm("");//确认
	
	window.document//页面文档（全部内容标签），可能比可视区域、html标签高/矮
	window.onscroll
	window.scrollTo(left,top);//触发onscroll

	xx.querySelector("").innerText = "";//""内为css选择器字符串。返回值为dom对象==>标签本身（未找到指定结果返回null）。多个符合要求的标签仅返回第一个。非实时更新
	xx.querySelectorAll("");//返回值为满足条件的dom对象数组，未找到指定结果返回长度为0的空数组
	document.getElementById("");
	xx.getElementsByTagName("");//根据标签名寻找，返回dom类数组，实时更新

	document.createElement("")//创建一个标签
	document.documentElement//快捷获取html标签=document.querySelector('html')
	document.documentElement.clientHeight//获取浏览器可视区域
	document.body//快捷获取body标签
	document.onmousewheel = function() {};//监听鼠标滚轮事件
	
	xx.onfocus//获取焦点事件
	xx.onblur//获取失去焦点事件
	xx.oninput//实时捕捉表单内容改变
	
	/*
		双标记dom对象有：
			innerText属性，可操控双标记中间的文本内容
			innerHTML属性，可操纵双标记中间的HTML代码
			
		所有dom对象有：
			onclick属性，绑定函数，浏览器调用，绑定时函数名不加括号。享受this权利，指向绑定元素本身。
			style属性，可操纵标签css样式
			className属性，可操纵标签class
			parentNode属性，获取父节点，html.parentNode=document
			parentElement属性，获取父节点，html.parentElement=null
			childNodes属性，包括文本元素节点
			children属性
			nextElementSibling/previousElementSibling属性，获取下一个兄弟节点/前一个兄弟
			classList.add/remove/contains/toggle[如果存在则移除，反之添加]/replace(a, b)
			tagName属性，可获取标签姓名（大写）
			nodeName属性，可获取标签类型（大写）
			appendChild()，可追加子元素
			getBoundingClientRect().top/height，详见0803
			setAttribute()，可设置属性值
			
		表单元素有：
			disabled属性，可控制是否禁用
	*/
```
