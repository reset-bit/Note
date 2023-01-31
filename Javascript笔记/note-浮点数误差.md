[toc]

# 0.1+0.2 ！== 0.3

js中，使用64位双精度浮点数表示数字。64位=>1位符号位+11位阶码+52位尾数。

`0.1`转为二进制表示，不断*2，结果若大于1则该位为1，直到得0。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200721114626622.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pIZ29nb2dvaGE=,size_16,color_FFFFFF,t_70)

故0.1二进制表示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200721114645122.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pIZ29nb2dvaGE=,size_16,color_FFFFFF,t_70)

由于有限的尾数，52位后的部分将发生舍入，1进位，0舍去。故`0.1`发生向上舍入，得到的数字比真实数字大。`0.2`同理。

二者相加同样超出52位，且53位为1，结果存储的时候仍然会发生向上舍入。

故，`0.1+0.2`发生了3次向上舍入。

解决方法：把计算数字提升10的N次方倍再除以10的N次方。`(0.1*1000 + 0.2*1000)/1000 === 0.3`

> http://www.fly63.com/article/detial/165



# 精确四则运算

## 乘法
```javascript
// 将小数变为整数来处理
function accMultiply(a, b) {
    let f1 = 1, f2 = 1;
    if(String(a).indexOf('.') !== -1) {
        f1 = String(a).length - String(a).indexOf('.') + 1;
        f1 = Math.pow(10, f1);
        a = a * f1;
    }
    if(String(b).indexOf('.') !== -1) {
        f2 = String(b).length - String(b).indexOf('.') + 1;
        f2 = Math.pow(10, f2);
        b = b * f2;
    }
    return a * b / f1 / f2;
}
```