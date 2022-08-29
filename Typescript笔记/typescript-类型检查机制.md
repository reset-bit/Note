[toc]

# 类型检查机制

ts编译器在做类型检查时所秉承的一些原则，以及表现出的一些行为。能够帮助我们辅助开发，提高开发效率

## 类型推断

不需要指定变量类型（函数返回值类型），ts能够根据某些规则自动的推断出类型

### 基础类型推断

```typescript
let a = 1;// a => number
let b = [];// b => any[]
let bb = [1];// bb => number[]
let c = (x + 1) => x + 1;// x => number c-return => number
```

### 最佳通用类型推断

当需要从多个类型中推断类型时，ts会尽可能推断出兼容当前所有类型的通用类型

```typescript
let b = [1, null];// b => number | null联合类型
```

### 上下文类型推断

根据表达式从左到右推断类型，通常会发生在事件处理中

```typescript
window.onclick = e => {}// e => MouseEvent
```

### 类型断言

当ts类型推断不符合预期的时候，使用类型断言强行指定类型

```typescript
interface Foo {
    bar: number
}
let d = {} as Foo;// 不推荐，若未赋值bar属性不会报错
d.bar = 1;
let d: Foo = {};// 推荐，未赋值bar属性会报错
```

## 类型兼容性

当类型Y可以被赋值给类型X时，即X = Y，目标类型=源类型，可以说X兼容Y。允许了类型不同的变量相互赋值

ts类型检查原则：鸭式变形法

```typescript
let s: string = 'a';
s = null;// tsconfig.json strictNullChecks: false ==> string兼容null

// 接口兼容性
interface Point2D {
    a: any;
    b: any;
}
interface Point3D {
    a: any;
    b: any;
    c: any;
}
let point2d: Point2D = {a: 1, b: 2};
let point3d: Point3D = {a: 1, b: 2, c: 3};
point2d = point3d;// no error 【成员少的兼容成员多的】，更精确的赋值给更模糊的
point3d = point2d;// error

// 函数兼容性
// 通常发生在函数相互赋值的情况下，如函数作为参数传递、函数重载。==>往少了进，往多了出
// 函数参数相互赋值的情况叫做函数参数的相互协变
// 条件1-参数个数
// 参数个数固定
type Handler = (a: number, b: number) => void;// 目标类型
function hof(handler: Handler) {return handler;}
let handler1 = (a: number) => {};
hof(handler1)// no error 【成员多的兼容成员少的】，多余参数认为undefined
let handler2 = (a: number, b: number, c: number) => {};
hof(handler2)// error 
// 参数个数不定
let a = (p1: number, p2: number) => {};
let b = (p1?: number, p2?: number) => {};
let c = (...args: number[]) => {};
a = b;// no error 固定参数兼容可选参数、剩余参数
a = c;// no error
b = a;// error tsconfig.json strictFunctionTypes: false ==> no error
b = c;// error
c = a;// no error 剩余参数兼容固定参数、可选参数
c = b;// no error
// 条件2-参数类型
let handler3 = (a: string) => {};
hof(handler3)// error
let p2d = (point: Point2D) => {};
let p3d = (point: Point3D) => {};
p2d = p3d;// error
p3d = p2d;// no error 【成员多的兼容成员少的】
// 条件3-返回值类型
let f = () => ({name: 'Alice'});
let g = () => ({name: 'Alice', location: 'Beijing'});
f = g;// no error 【成员少的兼容成员多的】
g = f;// error

// 枚举兼容性
enum Fruit {Apple, Banana};
enum Color {Red, Yellow};
let fruit: Fruit.Apple = 3;// no error 枚举与数字类型完全兼容
let number: number = Color.Red;// no error
let color: Color.Red = Fruit.Apple;// error 枚举类型之间不兼容

// 类兼容性
// 静态成员和构造函数不参与比较
class A {
    constructor(p: number, q: number) {};
    id: number = 1;
    private name: string = '';
}
class B {
    static s = 1;
    constructor(p: number) {};
    id: number = 2;
}
let a = new A(1, 2);
let b = new B(1);
aa = bb;// error 具有private属性，只有父类与子类才能相互兼容
bb = aa;// no error
class CC extends AA {}
let cc = new CC(1, 2);
aa = cc;// no error

// 泛型兼容性
interface Empty<T> {
    value: T
}
let e1: Empty<number> = {};
let e2: Empty<string> = {};
e1 = e2;// error 当泛型未在接口中使用时为no error
// 泛型函数
let log1 = <T>(x: T): T => {return x;}
let log2 = <U>(x: U): U => {return x;}
log1 = log2;// no error 定义相同并且没有指定类型参数
```

