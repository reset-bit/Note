[toc]


# redux(->vuex)

Redux 是 JavaScript 应用的状态容器，提供可预测的状态管理。

**整体思想**：
应用的整体全局状态以**对象树**的方式存在于store中，组件可以通过派发（dispatch）行为（action）给store，而不是通过其他组件。组件内部通过订阅store中的state来刷新自己的view。

**概念解释**：
1. redux中不能直接操作state，提供action对象来告诉store需要改变state。action对象中包含了state改变的具体信息：type属性必须，表示action name；payload属性表示携带信息。action creator是一类函数，可以通过传入参数的不同来动态生成action。
2. 一个state对应一个view。store.dispatch(action)是view发出action的唯一方法。
3. store收到action以后必须给出一个新的state，新state的计算过程叫reducer。reducer必须是一个纯函数，接收当前state和action作为参数，返回新的state。reducer因可以作为数组reduce()的参数而得名。

> 什么时候使用redux：
>
> - 有很多数据随时间而变化
> - 希望状态有唯一确定的来源
> - 将所有状态放在顶层中已不可维护

```jsx
// yarn add redux 安装redux依赖
// yarn add react-redux 连接组件与仓库
// yarn add redux-thunk 扩展dispatch方法，令其支持函数参数+异步。该参数函数接收dispatch与getState作为参数。

// store/index.js 仓库总控文件
import {creatStore,applyMiddleware,combineReducer} from 'redux';
import thunk from 'redux-thunk';
import address, {actionCreators as addressActions} from './address.js';// 名别名。address=>addressReducer
export {addressActions};// 非键值对简写，统一导入导出子仓库actionCreators
// 把数据分发给子仓库处理，将返回的数据重新合并成一个reducer
const reducer = combineReducer({ address });
// 接收一个reducer函数，该函数应当返回一个对象作为存储的数据
const store = createStore(reducer, applyMiddleware(thunk));// ori-reducer，以后每次组件dispatch都会调用reducer，据此修改子仓库数据
export default store;


// store/address.js 子仓库文件
// 关于try-catch：统一在发ajax时try/统一在仓库try。需要注意组件可能有收尾操作，组件try时需要仓库返回promise，async天生返回promise
// 枚举命名Pascal命名法，键名全大写需互斥，键值随意但建议具说明性，命名面向reducer
const ActionTypes = {
    ADDRESS_INIT: 'ADDRESS_INIT',
    ADDRESS_ADD: 'ADDRESS_ADD'
};
// 命名可小写。actionCreator执行后返回action对象，让组件不必关心action对象的具体创建，只需调用相应的creator函数即可。
// actions.type仅应当面向address子仓库，所以抽象actionCreators解耦
export const actionCreator = {
    // 异步闭包，柯里化将参数延迟到dispatch执行。async函数将作为dispatch参数，在该函数内调用dispatch。同步函数返回action
    // 或许在dispatch中执行了一次await
    init: () => async (dispatch, getState) => {
        let { address: {isInit} } = getState();
        if(isInit) return;
        try {
            let list = await http('/address/list');
            dispatch({ type: ActionTypes.ADDRESS_INIT, payload: list });
        } catch(e) {}
    },
    // 在toProps时统一dispatch
    add: playload => ({ type: ActionType.ADDRESS_ADD, payload })// 同步代码
};
/**
 * @param state 只读，上次返回的state数据
 * @param action 只读，含type属性用于对标操作，其他属性用于传递数据
 * 处理数据，更改redux中数据状态
 * 不能包含异步操作，必须是纯函数：相同的输入得到相同的输出
 * */
const initState = {list:[], isInit: false};// 设置参数默认值当做仓库初始值
const reducer = (state = initState, action: {}) => {
    let {type, payload} = action;
    switch(type) {
        case ActionTypes.ADDRESS_INIT:
            return {list:payload, isInit:true};// 应当重新复制一份数据，将setState转为return
        case ActionTypes.ADDRESS_ADD:
            return {...state, list:[...state.list, payload]};
        default:
            return state;// 未命中证明路由错误或首次数据初始化，应当返回原有值
    }
};
export default addressReducer;


// 从redux中获取数据到组件

// index.jsx 项目入口文件
// 使用Provider注入store
...
import {Provider} from 'react-redux';
import store from './store';
reactDom.render(
    <Provider store={store}>
    	<HashRouter><App /></HashRouter>
    </Provider>,
    document.getElementById('root')
);


// Address/Address.jsx 组件容器组件
// 容器组件负责获取数据，所以需要传递数据给展示组件。
// connect会在父组件与子组件之间添加connect层，并不会改变它连接的组件，而是提供一个经过包裹的connect组件。该组件中在mapStateToProps、mapDispatchToProps暴露props（props传递劫持）
// 数据同步更新。
import {connect} from 'react-redux';
import {addressActions} from '../../store';
import UIAddress from './UIAddress';
/**
 * @param state address子仓库中state
 * @param props 父组件传给本组件的props
 * @return {} 合并进props的属性
 * */
const mapStateToProps = (state, props) => {
    return {
        list: state.address.list
    };
};
/**
 * @param dispatch 显式提交事件函数
 * @param props 父组件传给本组件的props
 * @return dispatch返回的promise，可使用await获取promise中注入的数据
 * */
const mapDispatchToProps = (dispatch, props) => {
    return {
        init: () => dispatch(addressActions.init())
    };
};
export default connect(mapStateToProps, mapDiapatchToProps)(UIAddress);// 不需要某一参数时可传null


// Address/UIAddress.jsx 组件展示组件
class UIAddress extends Component {
    componentDidMount() {this.props.init();}// 在Address/UIAddress.jsx中connect UIAddress，故可以直接使用props
}
```

## redux调试工具安装

```js
// index.js
import { createStroe, applyMiddleware,compose } from 'redux';
...
const store = createStore(reducer, compose(applyMiddleware(thunk),
              	window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()));
export default store;
```

# isomorphic-fetch(->axios)

## axios与fetch

axios是一个基于Promise用于浏览器和node.js的http客户端，本质上是对XMLHttpRequest 对象的封装，只是Promise的实现版本。react可用axios。

```js
axios({
    method: 'post',
    url: '/user/login',
    headers: { 'Content-Type': 'application/json' },
    data: { user_name: 'zhangsan', user_pwd: '123' }
})
    .then(response => console.log(response))
	.catch(error => console.log(error))

// jquery
$.ajax({
    type: 'post',
    url: '/user/login',
    headers: { 'Content-Type': 'application/json' },
    data: { name, pwd },
    success: res => {}
});
```

fetch()用于在 JavaScript 脚本里面发出 HTTP 请求，可通过`window.fetch()`调用。

优势：语法简洁，更加语义化，对state/request/response等进行封装；基于标准promise实现，支持async/await；同构方便；更加底层，脱离XHR，API丰富

缺点：层次太低，需要封装；只对网络请求报错，400、500当做成功请求；默认不会带cookie，设置`fetch(url, {credentials: 'include'})`

```js
fetch('/user/login', {
    method: 'post',
    headers: { 'Content-Type': 'application/json' },// 默认text-plain文本
    body: JSON.stringify(this.state)
})// 承诺在此永不报错，默认get
	.then(response => return response.json())// 手动由二进制流对象转换为json对象，另可response.text()转换为字符串。
	.then(myJSON => console.log(myJSON))
```

## isomorphic-fetch封装

```js
import fetch from 'isomorphic-fetch';
export default (url, userOptions = {}) => {
    // 合并headers与json化
    let defaultHeaders = {
        'Content-Type': 'application/json',
        'Authorization'：sessionStorage.getItem('token')
    };
	userOptions.headers = Object.assign({}, defaultHeaders, userOptions.headers || {});
	userOptions.body && (userOptions.body = JSON.stringify(userOptions.body));
	return fetch(url, userOptions)
		.then(response => {
        	if(response.status === 200) return response.json();
        	else throw new Error(response.statusText);
	    })
		.then(({code, data, msg}) => {
        	switch(code){
                case 200:
                    return data;
                case 199:
                case 401:
                case 404:
                case 500:
                    throw new Erro(msg);
            }
	    })
		.catch(error => {
        	console.log(error.message);
       		return Promise.reject(error.message);
	    });
};
```

