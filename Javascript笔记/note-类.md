# 类

在ES6中，class (类)作为对象的模板被引入，可以通过 class 关键字定义类。

class 的本质是 function，可以看作一个语法糖，让对象原型的写法更加清晰、更像面向对象编程的语法。方法本质还是定义在prototype上。

```js
// 匿名类
let Example = class {
    constructor(a) {// 创建类的实例化对象时调用
        this.a = a;
    }
}
// 命名类
let Example = class Example {
    constructor(a) {
        this.a = a;
    }
}
// 类声明
// 不可重复声明，声明不可提升
class Example {
    constructor(a) {
        this.a = a;
    }
}

class Example = {
    static a = 2;// 静态属性：直接定义在类中的属性，无需实例化
    static sum(a, b) { return a + b }// 静态方法

	a = 2;
    constructor() {
        console.log(this.a);// 实例属性：生成实例后访问，不同的
        this.sum = (a, b) => { return a + b; }// 实例方法
    }

	sum(a, b) { return a + b; }// 原型方法：生成实例后访问，相同的访问地址
}
Example.a = 2;// 静态属性
Example.sum = (a, b) => a + b;// 静态方法
Example.prototype.a = 2;// 原型属性
Example.prototype.sum = (a, b) => a + b;// 原型方法
```

