[toc]

# 高级类型

## 交叉类型与联合类型

交叉类型：将多个类型合并为一个类型，新类型将具有所有类型的特性==>并集

联合类型：声明的类型并不确定，可以为多个类型中的一个

```ts
// 交叉类型
interface DogInterface {
    run(): void
}
interface CatInterface {
    jump(): void
}
let pat: DogInterface & CatInterface = {
    run() {};
    jump() {}
}

// 联合类型
// 字面量联合类型，限制变量取值在特定范围内
let str: 'a' | 'b' | 'c';
// 对象联合类型，在类型未确定的情况下，只能访问所有类型的共有成员
class Dog implements DogInterface {
    run()
    eat()
}
class Cat implements CatInterface {
    jump()
    eat()
}
enum Master {Boy, Girl};
function getPet(master) {// 依据传入参数判断返回不同类型的pet实例
    let pet = master === Master.Boy ? new Dog() : new Cat();// 被推断为dog与cat联合类型
    pet.eat();
    return pet;
}

// 可区分的联合类型
// 一个类型，如果是多个类型的联合类型，并且每个类型间有相同的公共属性，则能够根据该公共属性创建不同的类型保护区块。本质上是结合交叉类型与联合类型的类型保护方法
interface Square {
    kind: 'square',
    size: number
}
interface Rectangle {
    kind: 'rectangle',
    width: number,
    height: number
}
interface Circle {
    kind: 'circle',
    radius: number
}
type Shape = Square | Rectangle | Circle;
function area(s: Shape) {
    switch(s.kind) {
        case 'sqare': 
            return size * size;
        case 'rectangle':
            return width * height;
        case 'circle':
            return Math.PI * radius * radius;
        default:
            return ((e: never) => {throw new Error(e)})(s);// 检查s是否为never类型
    }
}
console.log(area({kind: 'circle', radius: 1}));
```

## 索引类型

实现对对象属性的查询和访问

```tsx
let obj = {
    a: 0,
    b: 1,
    c: 2
};
// 抽出指定对象的属性形成数组
function getValues(obj: any, keys: []) {
    return keys.map(key => obj[key]);
};
console.log(getValues(obj, ['a', 'b']));
console.log(getValues(obj, ['e', 'f']));// no error but want error

// 索引查询操作符 
// keyof T 类型T的所有公共属性的字面量的联合类型
interface Obj {
    a: number,
    b: number
}
let key: keyof Obj;// 'a' | 'b'
// 索引访问操作符
// T[K] 对象T的属性K所代表的类型
let value: Obj['a'];// typeof value === number

// T用于约束obj；另定义K并存在泛型约束-K必须从T的属性中取值，即K是T的所有属性的字面量的联合类型；返回值为数组，数组元素类型为K对应的类型
function getValues<T, K extends keyof T>(obj: T, keys: K[]): T[K][] {
    return keys.map(key => obj[key]);
}
```



## 映射类型

本质上是预先定义的泛型接口，通常会集成索引类型，从而将对象映射成希望的结构

```tsx
interface Obj {
    a: number,
    b: number,
    c: number
}

// 同态类型，只对自身属性约束
// readonly内置类库/映射类型：将所有Obj属性变为只读
type ReadonlyObj = Readonly<Obj>;
// partial：将所有Obj属性变为可选
type PartialObj = PartialObj<Obj>;
// pick:抽取Obj属性成为子集
type PickObj = PickObj<Obj, 'a' | 'b'>;

// 非同态类型，引入其它属性
// record:指定属性类型为已知类型
type RecordObj = RecordObj<'x' | 'y', Obj>;// Record {x: Obj, y: Obj}
```



## 条件类型

由条件表达式所决定的类型

```tsx
type TypeName<T> = 
	T extends string ? 'string' :
	T extends number ? 'number' : 'object';
type T1 = TypeName<string>;// string
type T2 = TypeName<string[]>;// object
type T3 = TypeName<string | string[]>;// string | object

// never | 'a' ==> 'a'
type Diff<T, U> = T extends U ? never : T;// ==>Exclude<T, U> 从T中去除可以赋值给U的类型
type T4 = Diff<'a' | 'b' | 'c', 'a' | 'd'>;// 'b' | 'c'
type NotNull<T> = Diff<T, undefined | null>;// ==>NonNullble<T>
// Extract<T, U> 从T中抽取出可以赋值给U的类型
type T5 = Extract<'a' | 'b' | 'c', 'a' | 'd'>;// 'a'
// ReturnType<T>  获取函数返回值的类型
type T6 = RetrunType<() => string>;// string
```

