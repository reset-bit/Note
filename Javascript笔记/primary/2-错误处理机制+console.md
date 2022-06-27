# Javascript·语法专题
[toc]
## 1.错误处理机制
### 1.1Error及其派生对象
javascript解析或运行时，一旦发生错误，引擎即抛出错误对象。程序将中断在发生错误的地方，不再向下执行。
Error对象必须有message属性，表示提示信息。

派生对象：
1. SyntaxError：语法错误
2. ReferenceError：引用不存在变量错误
3. RangeError：值超出有效范围错误。主要有几种情况，一是数组长度为负数，二是Number对象的方法参数超出范围，以及函数堆栈超过最大值
4. TypeError：变量或参数不是预期类型错误
5. URIError： URI 相关函数参数不正确错误

自定义错误：（继承Error）
```javascript
var err = new Error('出错了');
err.message;//出错了

function UserError(message){
	this.message = message || '默认信息';
	this.name = 'UserError';
}
UserError.prototype = new Error();//继承
UserError.prototype.constructor = UserError;
```
### 1.2throw && try...catch
程序遇到throw将终止，可抛出任何值

finally语句即使在try...catch中有return语句也依旧会执行

## 2.编程风格
- 全局变量对任何一个代码块都可读可写，使用大写字母表示变量名
- 由于变量提升，建议将变量声明放在代码块头部
- 使用严格相等运算符===
- 自增与自减使用+=和-=代替
- switch结构使用函数结构代替
```javascript
var actions = {
	'hack':function(){return 'hack';},
	'slash':function(){return 'slash';}
};
if(typeof actions[action] !== 'function'){
	throw new Error('invalid action');
}
//相当：
switch (action) {
    case 'hack':
    	return 'hack';
    case 'slash':
    	return 'slash';
    case 'run':
    	return 'run';
    default:
    	throw new Error('Invalid action.');
}
```

## 3.console
```javascript
console.log();//%c可输出css代码的参数
console.info();//相较于log在输出信息的前面加蓝色图标
console.warn();//黄色三角
console.error();//红叉
console.dir();//以易于阅读和打印的格式显示对象

console.table();//表格显示
console.count();//输出调用次数

console.assert(tfalse, '');//判断条件是否成立，false输出参数2，否则不会有任何结果

console.time('timer1');
console.timeEnd('timer1');

console.group('group1');//显示信息分组
console.groupEnd('group1');
```