[toc]

# 数组

特殊的对象，键名是按次序排列的一组整数。一个值总是先转成字符串，再作为键名进行赋值。支持多维数组。

## `.length`
`.length`max=2<sup>32</sup> - 1，可写。键名不需要连续，length总是=max键+1。
数组键名支持更改，若超出范围，该键名自动转为字符串。

```javascript
//var a = new Array();
var a = [1];
a['p'] = 'abc';//a.length=1，仅有一个整数键，不影响length属性
a[-1] = 'a';//a.length=1
a.foo = true;//a.length=1

Object.keys(a)//[ '0', 'p', '-1', 'foo' ]

a['p']//'abc'
a[-1]//'a'

delete a[0];//a.length=1，a[0]=undefined，hole
```

## 遍历方法
1. in检查某个【键名】是否存在于数组中（空位也返回false）
2. for-in循环会遍历到非数字键，故不推荐使用。可使用for/while循环遍历数组，或forEach方法。
> for-in、forEach、Object.keys皆跳过空位（不跳undefined）

```javascript
var a = ['red', 'green'];

2 in a//false

a.forEach(function (a){
	console.log(a);
});
```
性能：`for>for-of>forEach>map>for-in`

1. `arr.forEach()`方法对数组的每个元素执行一次给定的【函数】

  > `array.forEach(function(currentValue, index, arr), thisValue)` thisValue在function内部使用

2. `for-in`，用于对**对象{}的属性**以任意顺序进行循环操作。`for(var i in arr) {}`中i是数组下标

3. `for-of(ES6)`，用于对可迭代对象（仅实现了`Symbol.iterator`属性的对象）要迭代的数据进行循环操作。可迭代对象包括`Array/Map/Set/String/arguments`等，除此之外还可遍历dom对象集合

   > ```js
   > let obj = { test: 'test property' };
   > // for(val of obj) { console.log(val); }// error
   > obj[Symbol.iterator] = function* () {
   >         for(key in this) {
   >             yield this[key];
   >         }
   > };
   > for(val of obj) {
   >    	console.log(val);
   > }
   > ```


## 常用方法

```javascript
pop();// 删除数组最后一个元素并返回，如果数组为空返回undefined
push();// 将一个或多个元素添加到数组的末尾，返回值为数组最新长度
shift();// 删除数组第一个元素并返回，如果数组为空返回undefined
unshift();// 将一个或多个元素添加到数组的开头，返回值为数组新长度

forEach(function(item[, i]){}, object);// 对数组元素进行迭代，调用目标函数:this指向window;第二参数为元素下标。object指代forEach()中this指向。总是返回undefined
map(function() {});// 按照原始数组顺序依次处理元素，返回值为新数组。不会对空数组进行操作
reduce(function(prev, c) {}, ori);// 详见note-高阶函数

splice(startPos, num, replaceItems);// 删除/替换数组元素，返回值为包含被删除项目的新数组，没有或添加元素返回[]。num=0可用于在指定位置添加元素
slice(startPos, endPos);// 截取数组，返回值为新数组
concat(arr);// 拼接数组，返回值为新数组（不会改变原数组）
join(divider);// 把数组中所有元素放入字符串，可指定divider分割

// 查找指定元素在数组中位置，多个满足条件元素默认返回第一个
indexOf();// 找不到返回-1，否则返回从0开始的下标位置。适用于简单数据类型数组
lastIndexOf();
findIndex(function() {});// 相较于indexOf()适用性更广，返回值同indexOf()
includes(tar, fromPos);// 判断数组中是否包含该元素，es6。返回值为boolean。fromPos超出数组长度返回false。！支持类数组对象

find(function() {});// 按照条件查找返回元素，找不到返回undefined
filter(function() {});// 按照条件查找多个元素，返回新数组，找不到返回[]

every(function() {});// 判断每一个数组元素是不是都满足条件
Array.prototype.every = function(fun, value) {
    if (typeof fun !== 'function') {
        return false;
    }
    let arr = this;
    for(let i = 0; i < arr.length; ++i) {
        if(!fun.call(value, arr[i], i, arr)) {
            return false;
        }
    }
    return true;
};
some(function() {});// 判断数组元素有没有满足条件的

fill(target, start, end);// 填充数组为
```

## 类数组对象转数组
```javascript
var arr = Array.prototype.slice.call(arrayLike);

//利用call()将forEach()嫁接到类似数组的对象arrayLike上使用，比原生forEach要慢
function print(value, index){
	console.log(index + ':' + value);
}
Array.prototype.forEach.call(arratLike, print);
```

## 求max

```javascript
var arr = [6, 4, 1, 8, 2, 11, 23];

// 1-原始循环：
var result = arr[0];
for(var i = 1; i < arr.length; i++){
	result = Math.max(result, arr[i]);
}
console.log(result);

// 2-reduce方法：
function max(prev, next){
	return Math.max(prev, next);
}
console.log(arr.reduce(max));

// 3-sort方法：
arr.sort(function(a,b){return a - b;});//升序，b-a降序
console.log(arr[arr.length -  1]);
// （sort方法用于对数组的元素进行排序。排序顺序可以是字母或数字，并按升序或降序。默认排序顺序为按字母升序。）

// 4-eval函数：
var max = eval("Math.max(' ' + arr + ' ')");//max括号内为双单引号
console.log(max);
// （eval函数内，若为语句执行语句，参数为字符串。此处发生隐式类型转换：
// console.log(arr + ' '); //6,4,1,8,2,11,23
// 相当于：eval("Math.max(6,4,1,8,2,11,23)");//加空串转为字符串）

// 5-apply函数：
console.log(Math.max.apply(null, arr));
//（function.apply(obj,arr)，obj代替function内对象，arr传参给function数组）

// 6-ES6：
console.log(Math.max(...arr));
// （...扩展运算符，可以将一个数组变为参数序列）
```

## 扁平化

```javascript
function flatten(arr, result = []) {
    for(let item of arr) {
        if(Array.isArray(item))
            flatten(item, result);
        else 
            result.push(item);
    }
    return result;
}
// 或
function flatten(arr) {
    return arr.reduce((flat, toFlatten) => {
        return flat.concat(Array.isAray(toFlatten) ? flatten(toFlatten) : toFlatten);
    }, []);
}
```

