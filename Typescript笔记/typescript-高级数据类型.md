[toc]

# 高级数据类型

## enum

枚举，一组有名字的常量集合，属性只读

```tsx
// 数字枚举
// 在运行环境下被编译成对象，既可以用枚举成员的名字索引，也可以用值来索引。实现原理：反向映射
enum Role {
    Reporter = 1,// Role.Reporter=1 default=0
    Developer,// Role.Developer=2
    Maintainer
};
// 编译结果
{
    1: "Reporter",
    2: "Developor",
    3: "Maintainer",
	"Reporter": 1,
    "Developor": 2,
    "Maintainer": 3
}
// js转换结果
"use strict";
var Role;
(function (Role) {
    Role[Role["Reporter"] = 1] = "Reporter";// 先将空对象Reporter属性赋值为1，后将下标1的值赋值为Reporter，console.log(Role["Reporter"] = 1)==>1
    Role[Role["Developer"] = 2] = "Developer";
    Role[Role["Maintainer"] = 3] = "Maintainer";
})(Role || (Role = {}));
console.log(typeof Role);// object

// 字符串枚举
// 不可以进行反向映射
enum Message {
    Success = "成功",
    Fail = "失败"
};
// js转换结果
"use strict";
(function (Message) {
    Message["Success"] = "\u6210\u529F";
    Message["Fail"] = "\u5931\u8D25";
})(Message || (Message = {}));

// 异构枚举
// 容易混淆，不建议使用
enum Answer {
    N,
    Y = "yes"
}

// 常量枚举
// 在编译阶段被移除
const enum Month {
    Jan,
    Feb
};
// js转换结果
"use strict";
// 当不需要对象，而需要对象的值的时候，可使用常量枚举
let month = [Month.Jan, Month.Feb];
```

```ts
// 枚举成员属性
enum Char {
    // const，常量枚举成员，在编译阶段计算出结果，以常量形式出现在运行时环境
    a,
    b = Char.a,// 对已有枚举成员的引用
    c = 1 + 3,
    // computed，需要被计算的枚举成员，保留到程序运行阶段进行计算。在computed成员后的属性一定要被赋予初始值，否则报错
    d = Math.random(),
    e = '123'.length
}

// 某些情况下枚举与枚举属性可以当做特殊类型使用，两种不同类型的枚举不能进行比较
// 未赋初始值或数字枚举，可以把任意的number赋值给类型声明为enum的变量，
enum E {a, b};
enum F {a, b};
let e: E = 1;// no error
let ee: E = 'hello';// error
let f: E = 1;
console.log(e === f);// error
let e1: E.a;
let e2: E.a;
console.log(e1 === e3);// no error
// 字符串枚举只能赋值已有变量
enum G {a='a', b='b'};
let g:G = G.a;
let gg:G.a = G.a;
```

```typescript
// ex
// 可读性、可维护性较差
function initByRole(role) {
	if(role === 1 || role === 2) {
        // do something
    } else if(role === 3 || role === 4) {
        // do something
    } else if(role === 5) {
        // do something
    } else {
        // do something
    }
}
enum Role {
    Reporter = 'doReporter',
    Developer = 'doDeveloper',
    Maintainer = 'doMaintainer'
}
function initByRole(role) {
    if(role === Role.Reporter || role === Role.Maintainer) {
        // do something
    } else if(role === Role.Developer) {
        // do something
    }
    // Role[role](); error
}
```

## 函数

```typescript
// 定义
function add1(a: number, b: number) {
    return a + b;
}
let add2: (a: number, b: number) => number;// 声明
type add3 = (a: number, b: number) => number;// 声明，类型别名，为函数起一个名字
interface add4 {// 声明
    (a: number, b: number): number
}

// 可选参数
function add5(a: number, b?: number){
    return b ? a + b : a;
}
// 默认值
function add6(a: number, b = 1, c: number) {
    return a + b;
}
add6(1, undefined, 3);// 1+1+3=5
// 剩余参数
function add7(a: number, ...rest: number[]) {
    return a + rest.reduce((pre, cur) => pre + cur);
}

// 函数重载：名称相同但形参个数、类型不同的一系列函数，称为函数重载
// 函数重载与返回值无关，优点是增强函数可读性
// ts要求先书写函数声明列表，之后在类型宽泛的版本中实现重载
function add8(...rest: number[]): number;
function add8(...rest: string[]): string;
function add8(...rest: any[]): any {
    let first = rest[0];
    if(typeof first === 'number') {
        return rest.reduce((pre, cur) => pre + cur);
    } else if(typeof first === 'string') {
        return rest.join('');
    }
}
```

## 接口

约束对象、函数、类的结构和类型

```typescript
// 定义对象
interface List {
    readonly id: number,// 只读属性
    name: string,
    age?: number,// 可选属性
    [x: string]: any,// 字符串索引签名：用任意的字符串索引List，可以得到任意结果
}
interface Result {
    data: List[]
}
function render(result: Result) {
    result.forEach(item => {
        console.log(item.id, item.name);
    });
}
let result = {
    data: [
        {id: 1, name: 'A', sex: 'male'},// no error 鸭子类型，只要满足接口的必要条件即可。若传入字面量则报错：render({data:[{...}]})
        {id: 2, name: 'B'}
    ]
};
// 绕过字面量类型检查
render(result);// 将对象赋值给变量
render({data:[{...}]} as Result);// 类型断言，明确告诉编译器当前变量是指定类型
render(<Result>{data:[{...}]});// 不建议使用的类型断言，react中有歧义

// 可索引类型接口：不确定一个接口中有多少个属性
// 数字索引接口
interface StringArray {
    [index: number]: string// 用任意数字去索引StringArray，都会得到string
}
let stringArray: StringArray = ['A', 'B'];
// 字符索引接口
interface Chars {
    [x: string]: string,// y: number error
    // 索引签名混用，后索引签名类型必须是前一种的子集（前一种类型能够对后一种类型兼容），因为js会进行默认类型转换
    [y: number]: string,// y: number no error
}
```

```typescript
// 定义函数
let add: (a: number, b: number) => number;// 与以下声明方式等价
interface Add {
    (a: number, b: number): number
}

let add: Add = (a, b) => a + b;
```

```typescript
// 组合类型，可定义混合接口
interface Lib {
    (): void;
    version: string;
    doSomething(): void;
}
function getLib() {
    let lib: Lib = (() => {}) as Lib;// 无类型断言会报错缺少属性
    lib.version = '1.0';
    return lib;
}
```

## 类

```typescript
// 类成员修饰符
// public 所有属性默认值，对所有可见
// protected 只能被类或子类调用，不能被实例调用
// private 只能被类自身调用，不能被子类或实例调用
// 可以在构造函数参数处添加修饰符，令参数自动变成实例属性
// static 只能通过类名来调用，可以被继承
class Dog {
    constructor(name: string) {
        this.name = name;// 类属性都是实例属性。实例属性必须有初始值，或者在构造函数中被初始化
    }
    name: string;
    age?: number;// 可选属性
    run() {};
    readonly legs: number = 4;// 只读属性，需要初始化
}

// 继承
class Husky extends Dog {
    // 以private修饰的构造函数，既不能被继承，也不能被实例化
    // 以protected修饰的构造函数，不能实例化，只能被继承，相当于只声明了一个基类
    private constructor(name: string, color: string) {
        super(name);
        this.color = color;
    }
    color: string;// 子类自己的属性
}

// 抽象类：只能被继承，不能被实例化，便于实现多态（父类抽象出方法，在多个子类中有不同的实现，运行时出现不同的结果）
abstract class Animal {
    abstract sleep(): void;// 抽象方法，明确知道子类中有其他的实现=>子类必须实现
}
let animal = new Animal();// error
class Cat extends Animal {}

// 返回this实现链式调用
class workFlow {
    step1() {return this;}
    step2() {return this;}
}
class MyFlow {
    next() {return this;}
}
new WorkFlow.step1().step2()
new MyFlow.next().step1().next()// 多态：调用.next返回的类型既可以是子类，也可以是父类
```

## 接口与类的关系

接口是对动作的抽象，类是对对象的抽象

一个人可以继承一个抽象类（不可能同时是生物与非生物），可以实现多个接口（吃饭、睡觉等）

```typescript
interface Human {
    name: string;
    eat(): void;
}
// 类实现接口的时候，必须实现接口的全部属性。接口只约束类的public成员
class Asian implements Human {
    constructor(name: string) {
        this.name = name;
    }
    name: string;
    eat() {};
    sleep() {}; 
}

// 接口间继承
interface Man extends Human {
    run(): void;
}
interface Child {
    cry(): void;
}
interface Boy extends Man, Child {}
let boy: Boy = {
    name: '',
    eat() {},
    run() {},
	cry() {}
};
// 接口继承类：相当于把类成员抽象出来，只有结构，包括所有public/protected/private成员
class Auto {
    state = 1
}
interface AutoInterface extends Auto {}
class C implements AutoInterface {
    state = 1
}
class Bus extends Auto implements AutoInterface{}// 不需要实现state，因为Auto子类自然继承state
```

## 泛型

不预先确定的数据类型，具体类型在使用时确定

能够避免多条函数重载、冗长的联合声明类型，增强代码可读性

```typescript
function log(value: string): string {
    console.log(value);
    return value;
}
// 新增需求：接收字符串数组并打印
// 1-函数重载
function log(value: string): string;
function log(value: string[]): string[];
function log(value: any) {
    console.log(value);
    return value;
}
// 2-联合类型
function log(value: string | string[]): string | string[] {
    console.log(value);
    return value;
}
// 新增需求：可接收任何类型的参数
function log(value: any) {// 能够实现，但是丢失了类型信息，即类型之间的约束关系，输入和返回值类型必须是一致的
    console.log(value);
    return value;
}
function log<T>(value: T): T{
    console.log(value);
    return value;
}
log<string[]>(['a', 'b']);// 调用
log(['a', 'b']);// 类型推断调用，推荐

// 泛型函数
type Log = <T>(value: T) => T;// 函数签名
let myLog: Log = log
// 泛型接口
interface Log {
    <T>(value: T): T
}
interface Log<T> {// 约束所有成员，实现时必须指定类型
    (value: T): T
}
let myLog: Log<number> = log;
interface Log<T = number> {// 或者接口中指定默认类型
    (value: T): T
}
let myLog: Log = log;
// 泛型类
class Log<T>{
    // 不能应用于类的静态成员
    run(value: T) {...}
}
let log = new Log<number>();
log.run(1);// only number
let log = new Log();
log.run('a');// 可不传类型参数，使用类型推断

// 泛型约束
interface Length {
    length: number
}
function log<T extends Length>(value: T): T{
    console.log(value.length);
}
log([1, 2]);
log('12');
```

