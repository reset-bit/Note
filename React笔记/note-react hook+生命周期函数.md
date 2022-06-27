[toc]

# hook

自`react v16.8`始，针对函数组件开发，不编写`class`也能使用`state`以及其他的`react`特性

> 函数式组件没有`this`关键字，没有生命周期函数
>
> **为什么要用函数式组件**：
>
> - 学习class必须了解javascript中this的工作方式，this与其他语言差距较大
> - class不能很好的压缩，并且会使热重载出现不稳定的情况
> - 相比于class，函数式组件语法更为简洁

## useState

`react`为每一个函数组件提供一个`state`容器，该容器是**数组**。想要往state容器中放值就要调用`useState()`，因此要求每次`useState()`数量和次序是相同的，即该函数不能放在条件判断分支中。基本类型保存数值，引用类型保存地址。**每次`render`，未更改的基本类型会重新创建，未更改的引用类型将不做改变。**

`state`容器内部维护`index`，`setXxx()`将闭包到`state`容器中应当指向的`index`下标，因此能够准确更改指定项的值（**总是返回同一个函数**）。

```jsx
import React, {useState} from 'react';
const Home = () => {
    // 调用setXxx()，整个组件将更新即重新渲染，该更新是异步的。函数组件的刷新其实是【函数】的反复执行，每份state相互独立。副本与原数据相比，地址不同，数据来源相同（state容器)。
    // 使用setXxx()更新引用类型，必须【提供一份不同的对象】，这里的对象是广义的对象。setXxx()的更新原理是【替换】，但class组件中setState()更新原理是合并。若更新的基本类型与原有数据相同，则不更新
    // 不建议在一段代码中调用setXxx()多次（即使是针对不同的变量），react并不会将多次setXxx()合并执行
    // setXxx()异步更新，第二参数为回调函数
    const [a, setA] = useState(0);
    const [person, setPerson] = useState({name: 'zhangsan', age: 18});
    return (
    	<div>
        	<span>{person.name}</span>
            <button onClick={() => setPerson({...person, name: 'lisi'})}>change name</button>
            <button onClick={() => {
                    // 适用更改数据项很多的情形
                	person.name = 'lisi';
                    setPerson({...person});
                }}>change name</button>
            <button onClick={() => setTimeout(() => console.log(person.name)), 3000)}>print variate after 3s</button>
        </div>
    );
};
```

```js
// 手写useState
let index = 0;
let state = [];// state容器
function useState(initValue) {
    let curIndex = index;
    index = index + 1;
    let value = null;
    // 基本类型直接赋值
    if(!Object.isObject(state[curIndex])) { value = state[curIndex] }
    // 引用类型返回拷贝
    else { value = Object.assign({}, state[curIndex]); }
    let set = function(newValue) {// 闭包curIndex->index
        if(newValue === state[curIndex]) return;// ===，引用类型判断地址
        state[curIndex] = newValue;
       	render();// 更新组件
    }
    return [value, set];
};
function render() {
    index = 0;// 重置index，重渲染组件
    component();
};
function component() {
    let [a, setA] = useState(100);
    return ({setA(101)});
};
```

## useEffect

`effect`副作用（生命周期函数中执行的任务，如`ajax`、产生/销毁计时器、订阅/取消订阅事件），实现了**关注点分离**。

主要是为了解决：

- 多个任务写在同一个生命周期函数中，令该函数内部代码关联性不高，逻辑混乱
- 同一个任务的不同部分散落在不同的函数中，看不出整体任务的逻辑和全貌

`react`为每一个函数组件在内存中提供`effect`任务队列，使用`useEffect()`会向队列中添加一个任务，添加顺序决定任务执行顺序。`react`将在**本轮dom更新完毕**后开始逐一执行副作用队列中的任务，在**下一轮dom更新完毕**后逐一清除副作用队列中的任务（若没有依赖数据，则在组件销毁时执行副作用函数）。相当于实现了`componentDidMount()/componentDidUpdate()/componentWillUnmount()`等

```jsx
// useEffect(fun[, arr])  fun: 副作用函数; arr: 依赖数据（useEffect将在依赖数据改变时执行，一定注意依赖数据完整性。不要在副作用函数中修改依赖数据，将造成死循环）
import React, {useState, useEffect} from 'react';
const Home = () => {
    const [a, setA] = useState(100);
    useEffect(() => {...}, []);// 页面初始化执行
    useEffect(() => {...});// 页面初始化和更新执行，相当于直接写...
    useEffect(() => {...}, [a]);// state.a改变后时执行，页面初始也会执行-->useRef
    // 如果在参数函数里占用了系统资源和内存（如setInterval()...），需要返回清除副作用函数把对占用的资源和内存进行释放。需要通过【route跳转】才能看到销毁信息。直接输入url相当于【刷新页面】，销毁之后瞬间跳转，看不到销毁信息
    useEffect(() => {// 副作用函数：依赖数据改变时
        setInterval(...);
        return () => {...};// 清除副作用函数：state/props数据改变->dom更新->副作用函数->state/props数据改变->dom更新->【上轮清除副作用函数】->副作用函数
    }, []);
    return (...);
};
```

## useRef

`react`为每一个函数组件维护了一个`ref`容器，返回一个引用的对象

用途：

- 绑定dom
- 将一些不大的、改变值不需要`re-render`（即，需要跨`render`共享）的数据放在ref中
- 子传父方法或属性
- 实现类似 class 组件挂载属性（vue-form）的效果

### 实现vue-ref

```jsx
// class
class Home extends Component {
    render() {
        return (
       		<div>
            	<input ref={el => this.inputRef=el;} type='text'/>
                <button onClick={() => this.inputRef.focus()}>get focus</button>
            </div>
        );
    }
}

// function
const Home = () => {
    let inputRef = useRef();// 放dom，初值为空。每次dom更新都指向指定的dom，ref容器内对象地址不变
    let numRef = useRef(1);
    return (
    	<div>
        	<input ref={inputRef} type='text' />
            <button onClick={() => inputRef.current.focus()}>get focus</button>
            {/* numRef将不会更改，组件也不会更新 */}
            <button onClick={() => numRef.current = 100}>change num</button>
        </div>
    );
}
```

### 避免初次渲染
```jsx
// 监听某一个数据的同时，在页面第一次渲染的时候不执行相应逻辑：设定flag标识是否初次渲染
const Home = () => {
    let isMountingRef = useRef(true);
    const [a, setA] = useState(100);
    useEffct(() => {
        if(isMountingRef.current) { isMountingRef.current = false; return; }
        ...
    }, [a]);
    useEffect(() => {isMountingRef = false}, []);// 第一次渲染后执行，在return中读取isMounting=false
    return (...);
};
```

### 子传父(useRef + forwardRef + useImperativeHandle)

```jsx
// Home.jsx
const Home = () => {
    let childRef = useRef();
    return (
    	<div>
        	<Child ref={childRef} />
            <button onClick={() => childRef.current.foo()}>child-foo</button>
        </div>
    );
};
// child.jsx
import React, {useState, forwardRef, useImperativeHandle} from 'react';
const Child = (props, ref) => {
    const [a, setA] = useState(100);
    const foo = () => {...};
    useImperativeHandle(ref, () => {// 返回一个对象。即父组件中的childRef.current
        return {a, foo};
    });
    return (...);
};
export default forwardRef(Child);// 令Child第二参数为ref
```

## 性能优化

> 已知`react`已经具备的性能优化方式：
>
> - 事件绑定自动冒泡处理

### memo()

高阶组件，支持组件记忆，实现类似`PureComponent`的功能。仅比较`prop`，若改变则`re-render`。

可以优化：未传props、props为**基本类型、引用-对象类型、引用-数组类型**。

react-router v6以后，URL参数使用useParams hook获取，不再算作props内。

**失效情况**：父组件给子组件添加事件、组件内部使用useState/useReducer/useContext等

```jsx
// Child.jsx
import React, {memo} from 'react';
const Child = () => {...};
export default memo(Child);
                     
// yarn add lodash（javascript工具包）
// 由于被路由直接渲染的组件将会携带很多参数在props中，这时使用isEqual()将会十分消耗性能。即，isEqual()使用场景需要具体问题具体分析(props:fa-ch + route + redux)。一般【仅优化展示组件】
// Child2.jsx
import React, {memo} from 'react';
import _ from 'lodash';
const Child2 = () => {...};
const isEqual = (prevProps, nextProps) => {return _.isEqual(prevProps, nextProps)};
export default memo(Child, isEqual);
                      
// Home.jsx
const Home = () => {
    const [c, setC] = useState([]);
    render() {<Child obj={{c, setC}}/>};// 每次重新创建obj，故Child2总是刷新。使用lodash isEqual()避免刷新
};

// Child.jsx
import React, {memo} from 'react';
const Child = () => {};
// 由路由直接渲染，且父组件传了arr
// 只有arr深比较，其它情况（如路由参数改变）浅比较
const isEqual = (prevProps, nextProps) => {
    let keys = Object.keys(prevProps);
    for(let i = 0; i < key.length; ++i) {
        if(key[i] === 'arr' && !_.isEqual(prevProps[key[i]], nextProps[key[i]])) {return false;}
        else if(prevProps[key[i]] !== nextProps[key[i]]) {return false;}
    }
    return true;
};
export default memo(Child, isEqual);
```

### useCallback

处理回调的副作用，可以优化：props为**引用-函数类型**（`useState-setXxx()`直接传递即可，返回的总是同一个函数；需要传递给子组件的函数建议使用`useCallback`，其它可不用）

```jsx
// useCallback(fun[, arr]) fun：业务逻辑函数; arr：依赖数据，用法同useEffect，总是闭包依赖数据
// Home.jsx
import React, {useCallback} from 'react';
const Home = () => {
    const foo = useCallback(() => {console.log('foo')}, []);// only make Home re-render
    <Child foo={foo}/>
};
```

## useReducer

为单个组件创建数据集中管理服务，掌管全局状态。`useState`代替方案。在某些场景下，`useReducer` 会比 `useState` 更适用，例如 state 逻辑较复杂且包含多个子值，或者下一个 state 依赖于之前的 state 等。

原生hook无法接收函数作为参数。

```jsx
// 强制刷新
import React, {useReducer} from 'react';
const Foo = () => {
    // const [state, dispatch] = useReducer(reducer, initialArg, init);
    const [ignored, forceUpdate] = useReducer(x => x + 1, 0);
    function hanldeClick() {
        forceUpdate();
    }
}

// other case
// yarn add react-hook-thunk-reducer 使用useThunkReducer()来扩展diapatch函数，使其允许接受函数作为参数来发ajax
// Cart.jsx
import React, {useReducer,useEffect} from 'react';
import useThunkReducer from 'react-hook-thunk-reducer';
import reducer, {actionCreator as cartActions} from './cartReducer.js';
const Cart = () => {
    const [state, dispatch] = useThunkReducer(reducer, {list: []});// state从reducer中拿数据使用，dispatch通知reducer数据发生改变
    useEffect(() => {
        dispatch(cartActions.init());
    }, []);
    return (...);
};
// cartReducer.jsx
import http from '@/utils/http.js';
export default (state, action ) => {
    let {type, payload} = action;
    switch(type) {
        case 'CART_INIT':
            payload.forEach(item => {
                item.checked = true;
                item.checkedEdit = false;
            });
            return {list: payload};
        default: return state;
    }
};
export const actionCreator = () => {
    init: () => async (dispatch, getState) => {
        let list = await http('/cart/list', {method:'post'});
        dispatch({type: 'CART_INIT', payload: list});
    }
};
```

## useMemo

对函数return的结果进行缓存，函数组件实现计算属性。组件初始化执行一次。

> 与useCallback的区别：useCallback是对整个函数进行缓存，绑定事件不会失效
>
> 类组件实现计算属性：yarn add memorize-one

```jsx
// Home.jsx
import React, {useState, useMemo} from 'react';
const Home = () => {
    const [a, setA] = useState(1);
    const [b, setB] = useState(2);
    const sum = useMemo(() => {
        return a + b;
    }, [a, b]);
    return (<span>{sum}</span>);
};
```

## useLayoutEffect

每次dom构建完毕，渲染之前执行。可以拿到最新数据对应的dom进行相应的属性更改

不能包含异步代码，因为`useLayoutEffect`会阻塞页面dom渲染，影响用户体验。优先使用`useEffect`

> `useEffect`的执行时机是：dom渲染完毕之后执行。若使用`useEffect`代替`useLayoutEffect`，则会出现dom闪烁的情况（渲染之后再次渲染）

```jsx
// Home.jsx
import React, {useState, useRef, useLayoutEffect} from 'react';
const Home = () => {
    const divRef = useRef();
    const [a, setA] = useState(100);
    useLayoutEffect(() => {
        divRef.current.style.marginLeft = `${a}px`;
    }, [a]);
    return (
    	<div ref={divRef} className={style['demo']}>
        	{a}
            <button onClick={() => setA(a + 100)}>change a</button>
        </div>
    );
};
// Home.module.styl
.demo
	width: 100px
    height: 100px
    background-color: gray
```

## useContext

跨组件的数据传递，从context对象中存取数据

```jsx
// Home/context.js
import {createContext} from 'react';
export default createContext();

// Home/Home.jsx
import context from './context.js';
const Home = () => {
    return (
    	<context.Provider value={{key:1}}>
        	<Child />
        </context.Provider>
    );
};

// Home/Child.jsx
import React, {useContext} from 'react';
import context from './context.js';
const Child = () => {
    let value = useContext(context);// 获取上下文数据对象
    return (
    	<div>{key.value}</div>
    );
};
```

# 生命周期函数

只有类组件才有生命周期函数，函数式组件使用hook

```js
class Profile extends Component {
    // 挂载
    constructor(props) {super(props);}// 初始化state、为方法bind this。super()为通过调用父类构造方法创建父类实例，若state中使用到了props则需要写参数props。
    componentWillMount() {}// 【已废弃】
    static getDerivedStateFromProps(nextProps, prevState) {}// derived派生。使用本函数必须要定义state，return( null)返回值将合并到state中，支持state覆盖。静态函数决定没有this关键字。应用场景如父组件给子组件传值，该值需要初始化子组件state，以解决父子组件单向数据流问题。参数为即将生效的props和马上失效的state。只建议使用props，不建议赋值st，更新次数太频繁（在创建和更新时都会调用）【不建议使用，componentWillMount平替】
    render() {}// 返回jsx对象，决定dom呈现
    componentDidMount() {}// 发ajax、开启定时器、开启事件监听
    
    // 更新
    // 1.组件内部setState(); 2.context改变（如果使用）; 3.路由等原因导致的props改变; 4.父组件导致的子组件更新：更新执行render()、props改变等
    componentWillReceiveProps(nextProps) {}// 父组件导致的子组件更新时调用。自定义对比props变化更新state【已废弃】
    // static getDerivedStateFromProp() {}
    shouldComponentUpdate(nextProps, nextState) {}// 自定义决定是否进行dom数据更新：return true/false; 数据更新一定会执行，返回true将执行componentWillUpdate()等，此时数据更新将延时到componentWillUpdate()之后执行；返回false将不执行componentWillUpdate等
    componentWillUpdate(nextProps, nextState) {}// 数据更新会与马上要进行的更新进行合并【已废弃】
    getSnapshotBeforeUpdate(nextProps, nextState) {}// 在更新dom前读取dom相关信息，通过return( null)将该值传递给componentDidUpdate()。应用场景如页面刷新前保存滚动条位置信息，刷新后重新定位等。【componentWillUpdate平替】
    // render() {}
    componentDidUpdate(prevProps, prevState, value) {}// 参数为刚刚失效的props、state，可与this.props、this.state进行数据对比，实现类似vue watch中的功能。建议进行dom操作，进行数据操作可能会导致死循环
    
    // 卸载
    componentWillUnmount() {}// 释放组件占用的资源和空间。如销毁计时器、销毁new关键字声明的变量、取消事件监听
}

/*
	父子组件生命周期函数执行顺序：
	// 挂载
	fa-constructor
	fa-componentWillMount
	fa-render
	ch-constructor
	ch-componentWillMount
	ch-render
	ch-componentDidMount
	fa-componentDidMount
	// 更新
	fa-shouldComponentUpdate
	fa-componentWillUpdate
	fa-render
	ch-componentWillReceiveProps
	ch-shouldComponentUpdate
	ch-componentWillUpdate
	ch-render
	ch-componentDidUpdate
	fa-componentDidUpdte
	// 销毁
	fa-componentWillUnmount
	ch-componentWillUnmout
*/
```