[toc]
# void

执行一个表达式，返回undefined。适用于超链接中防页面跳转。
```html
<!-- 用户点击链接提交表单，但不产生页面跳转 -->
<a href="javascript:void(document.form.submit())">文字</a>
```

# in

判断指定的属性是否在指定的**对象或其原型链**中。

```javascript
console.log('name' in Person);// true
```


　## 双等号==

先将字符串/布尔值进行类型转换，转为基本数据类型。主要调用`valueOf()、toString()`两种方法。前者优先级更高。

```js
false == ''// true，0 == 0
// 对null来说，除与undefined比较为true外，其他的都是false
null == undefined// true
null == false// false
null == 0 // false，0是数值，null类型为Null（与typeof无关）
[] == ![]// true: [] == false; []调用valueof()仍为[]，故调用toString(); '' == false; 0 == false;

null >= 0// true 先转换为数字
[] != []// true 引用数据类型，比较地址
```

![image-20210924210230267](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210924210230267.png)

# 位运算

js数值由64位浮点型表示，进行位运算时，自动转换为【32位有符号】整数，舍弃小数位。表示范围是-2<sup>32</sup>~2<sup>32</sup>-1。（max=01...11；min=10...00，即-0，表示最小的数）

## ~`not`

转32位->转二进制反码->二进制数转为浮点数 ===> 求负-1

如果运算元不是可用的整数，将取0作为运算元

```javascript
console.log('~null: ', ~null);       // => -1
console.log('~undefined: ', ~undefined);  // => -1
console.log('~0: ', ~0);          // => -1
console.log('~{}: ', ~{});         // => -1
console.log('~[]: ', ~[]);         // => -1
console.log('~(1/0): ', ~(1/0));      // => -1, 非可用整数
console.log('~false: ', ~false);      // => -1
console.log('~true: ', ~true);       // => -2, true=1
console.log('~1.2543: ', ~1.2543);     // => -2
console.log('~4.9: ', ~4.9);       // => -5
console.log('~(-2.999): ', ~(-2.999));   // => 1
```

# mod

取余（rem，%）和取模（mod）都是求商和余数的过程`c=a/b; r=a-(c*b);`。二者的不同之处在于负数的计算结果：

1. 计算商值：%向0靠近；mod向负无穷靠近
2. 计算余数：%正常求取；mod余数对模求余数，保证一定在模内

```javascript
function mod(n, m) {
    return ((n % m) + m) % m;
};
```

# 点操作符.与下标运算符[]

点操作符：静态的，右侧必须是一个以属性名称命名的简单标识符。该标识符必须直接出现在程序中，无法修改。

下标运算符：动态的，方括号里是一个计算结果为字符串的表达式，能够修改。

**即，下标运算符可以用变量作为属性名访问，点操作符不可以。**
