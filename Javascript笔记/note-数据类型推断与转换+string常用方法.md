# 数据类型推断与转换
基本数据类型：Number、String、Boolean、Null、Undefined（ES6: Symbol）
复杂数据类型：Object

```
Number:
	window.Infinity// 正无穷
		Math.max() = -Infinity;
		Math.min() = Infinity;
	NaN// 不是数字
	
String：
	// 本质是字符数组
	trim()
	split()// 分割字符串
	slice(startPos, endPos)// 字符串截取，返回新字符串，[,)。可以是负数，如-1取最后一个字母
	substring()// 截取数组，不接受负数参数
	indexOf(str, startPos)
	lastIndexOf()// 子字符串在整个字符串中最后出现的位置
	toUpperCase()// 返回新字符串
	toLowerCase()
	charAt()
	charCodeAt()// 返回指定位置字符的unicode编码，若index不正确，返回NaN
	replace(oldstr, newstr)// 字符串替换，返回新字符串，只会替换一次。正则表达式替换所有目标oldstr
	match()// 正则表达式复杂检索字符串
	str.split('').reverse().join('');// 字符串反转
	
Boolean:
	false：
		0 | NaN | "" | null | undefined

undefined:
	返回undefined:
		变量未定义；对象未赋值属性；数组未初始化元素或越界访问；函数调用时未传参的形参；未定义返回值函数的返回；void()返回值
	转为数字为NaN

null：
	typeof null// 返回object，因为null的本质是空对象指针;历史遗留问题，js开始使用32位系统，为了性能考虑使用低位存储，00开头表示对象。null表示全0，所以返回object
	转为数字为0

```
1. typeof
`typeof number / typeof(number)`，返回值全部为字符串：number | string | boolean | undefined | object | function

2. isNaN
`isNaN(NaN)`，返回值为boolean

3. === null

4. parseInt
只有包含数字才转化，否则转换为NaN

5. toString()/String()/Number()/Boolean()

6. 判断完全相等：+0与-0应为false，NaN与NaN想要为true（实际上由于NaN表示不是一个数字，结果情况很多，应当返回false。使用中常常想判断某一个值是不是NaN，所以想要为true）
```javascript
// 简单类型
function identity(val1, val2) {
	return Object.is(val1, val2);
}
// 对象，支持属性打乱顺序。若对象属性中存在嵌套结构，需要使用递归实现
const tar1 = {
  name:'张三',
  age:12,
  address:'上海市浦东新区'
}
const tar2 = {
  name:'张三',
  address:'上海市浦东新区',
  age:12,
}
const tar3 = {
  name:'张三',
  address:'上海市浦东新区',
  age:13,
}
// return true 相等
function foo(item1, item2) {
    const _item1 = JSON.stringify(item1);// 转换为字符串
	// 判断是否有新加属性
    const _item2 = JSON.stringify({...item1, ...item2});
    // const _item2 = JSON.stringify(Object.assign(item1, item2));
    return _item1 === _item2;
};
foo(tar1, tar2);// true
foo(tar1, tar3);// false
```

7. 非数值转换为数值
```javascript
parseInt()/parseFloat()// 针对字符串的转换更合理一些
1.找到第一个非空字符，若不是数字字符或者是负号，NaN
2.找到第一个非空字符，若是数字字符，但数字字符中间含有其他符号，只返回正常的数字
3.''/null/undefined/，NaN
4.parseInt()可指定转换基数

Number() 
1.数字值，简单的传入传出
2.Boolean值，0或1
3.null/''，0；undefined，NaN
4.字符串仅包含数字，忽略前导0与空格，转为十进制
5.字符串包含字符，NaN

1 + '1'// 2
100 > '99'// true
```

8. 字符串转对象、函数

```js
let obj = JSON.parse(str);
// 这里eval实际是解析了表达式(false || function test(value))
// eval可以解析任何字符串，这是不安全的，请尽量不要使用。
let funcStr = "function test(value){alert(value)}";
let test = eval("(false || "+funcStr+")");
test("函数能够执行");
```

9. parseInt(str, radix)
当radix值为0，或没有设置该参数，parseInt()会根据str来判断基数，此基数将作为转换时的依据。

- 如果以0x开头，将剩余部分解析为十六进制整数
- 如果以0开头，将剩余部分解析为八进制或十六进制的数字
- 如果以1~9开头，将其解析为十进制整数

10. toString(radix)
数字的字符串表示，radix=2时，表示二进制形式的字符串

# Date

日期类型
```js
var date = new Date();
var date = new Date(1971, 1, 1, 0, 0, 0);
    getYear();// 191
    getFullYear();// 2021
    getMonth();// 7, 8-1
    getDay();// 3，星期三
    getDate();
    getHours();// 24小时制
    getMinutes();
    getSeconds();
    getMilliseconds();
    getTime();// 距1970-01-01 00:00:00毫秒数，常用于图片命名
```

# object
随时动态开辟属性，灵活的判定赋值表达式
```js
var obj = { key1:1 };
delete obj.key1;// == delete obj['key1']
for(let k in obj) {// 遍历对象所有【键名】，k为字符串
    console.log(k);
    console.log(obj[k]);
}
```
