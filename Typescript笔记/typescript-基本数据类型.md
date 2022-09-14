[toc]

# 写在前面

ts和js相比主要做了那些改动？

- 类型检查。ts会在编译代码时进行严格的静态类型检查
- 语言扩展。包括es6以及未来提案中的特性，并借鉴其他语言中的某些特性
- 工具属性。ts能够编译成标准的js

---

javascript是一门动态弱类型语言，在运行时动态计算属性的偏移量，使用额外的空间存储属性名

> 动态类型：在程序执行阶段确定所有变量的类型
>
> 弱类型：可以任意改变一个数据类型的语言



# hello world

1. npm i typescript -g
2. npm init -y，创建package.json
3. npm i webpack webpack-cli webpack-dev-server -D
4. npm i ts-loader typescript -D
5. 处理配置文件（webpack.config.js/webpack.base.config.js/webpack.dev.config.js/webpack.pro.config.js）
6. npm i html-webpack-plugin -D，webpack.base.config.js
7. npm i clean-webpack-plugin -D，webpack.pro.config.js
8. npm i webpack-merge -D，webpack.config.js
9. 自定义npm start/npm run build命令



# 基本数据类型

js基本类型：string/number/boolean/symbol/null/undefined

   引用类型：array/function/object...

ts在此基础上新增：void/any/never/元组/枚举/高级类型

```ts
// 为变量赋值与声明类型不符的数据将会报错
// 原始类型
let bool: boolean = true;// 类型注解，相当于强类型语言中的类型声明，变量/函数:type
let num: number | string = 1;// 联合类型
num = '2';

// 数组
let arr1: number[] = [1, 2];
let arr2: Array<number> = [1, 2];// Array为ts提供的泛型接口
let arr3: Array<number | string> = [1, '2'];

// 元组，初始化时只能够严格符合声明类型与个数，可以使用push等方法，但push的数据无法访问
let tuple: [number, string] = [0, '1'];

// 对象
let obj1: object = {x: 1, y: 2};// obj1.x = 1报错，因为只声明对象类型，未指定属性类型
let obj2: {x: number, y: number};

// 函数，一般返回值不加类型注解也可，利用ts数据推断功能
let add = (x: number, y: number): number => x + y;
let compute: (x: number, y: number) => number;// 声明
compute = (a, b) => a + b;// 实现，将自动提示参数类型

// undefined null，任何其它类型的子类型
// 在tsconfig.json中开启strictNullCheck:false控制台将忽略不同类型间undefined、null赋值错误；或使用联合类型
let un: undefined = undefined;
let nu: null = null;

// symbol
let s1: symbol = Symbol();
let s2 = Symbol();
console.log(s1 === s2);// false

// void，声明没有任何返回值，undefined子类型
// undefined在js中不是保留字，可能会存在自定义undefined覆盖全局undefined情况。void运算符能够保证总返回undefined
// void在ts中作为返回类型能够被不同的类型替换，以此达成高级回调模式
let noReturn = () => {};
function dosomething(callback: () => void) {let c = callback();}// void兼容性保证不会报错。若void=>undefind/number/...，将报错
function callback(): number {return 2;}
dosomething(callback);// c=2

// any，不加类型注解时的默认类型，同js
let foo = 1;

// never，永远不会有返回值，如抛出错误、死循环
let error = () => {
    throw new Error('error');
}
let endless = () => {
    while(1) {}
}
```



子类型的值可以赋值给父类型的变量



task：类、枚举、函数、泛型、类型推论、类型兼容性