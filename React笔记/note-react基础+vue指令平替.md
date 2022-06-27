[toc]

# react基础

```js
(npx) create-react-app xiaomi-react// 1
npm init react-app xiaomi-react// 2
yarn creat react-app xiaomi-react// 3
cd xiaomi-react
yarn start// 端口默认3000

// 项目目录分析
- public// 静态目录文件夹，与src/assets区别是不会被webpack编译。src/assets缺少文件会报编译错误，但过大会导致打包出来的文件很大，从而导致首屏白屏时间过长。index.html访问使用%PUBLIC_URL%，js访问使用process.env.P
- src
	index.js// 入口文件
	App.jsx// 根组件
	App.css// 根组件样式
package.json
	dependencies// react: 17.0.2
   
// index.js
// react支持jsx语法（在js中直接写html代码，jsx对象），需要引入react。react dom在渲染所有输入内容之前，默认会进行转义。所有内容再渲染之前都被转换成字符串，可防止XSS攻击
import React from 'react';
import reactDOM from 'react-dom';
import App from './App.jsx';// 导入
reactDom.render(
	<App />,// 需要渲染的内容，必须使用pascal命名法，无需注册直接使用
    document.getElementById('root')// 根节点，不会被替换。比使用document.querySelector()效率更高
);

// App.jsx
// 定义组件的两种方式：类组件、函数组件
// 1-类组件
import React from 'react';
class App extends React.Component {
    // es6简写: render: function() {}
    render() {
        return (// 建议使用()包裹jsx对象
        	<div>this is App compnonent</div>
        );
    }
}
export default App;
// 2-函数组件
// 没有this关键字，没有生命周期函数
import React from 'react';
export default () => (
	<div>this is App compnonent</div>
);
```

> 1. 书写变形：`class->className; for->htmlFor`
> 2. 防止父组件样式污染：使用css module（从工具层面给出生成局部css的规范，css loader将会编译成hash字符串），默认开启，将.less/.css改为.module.less/.module.css，`className={style['aaa']}`
> 3. 修改antd默认样式：使用:global语法声明全局规则，这样声明的class不会被编译为hash字符串，`className={'aaa'}`
> 4. react页面在初次渲染时会特意渲染2次，目的在于让开发者注意到可能出现的副作用
> 5. 配置@，把一个路径或路径片段的序列解析为一个绝对路径： `alias:{ "@": path.resolve("src"),... }// config/webpack.config.js-329`
> 5. 组件强制刷新：在`state/props/context...`改变时

# vue指令平替

- `setState`：非立即（异步）执行，将一段同步代码内的`setState`自动合并，只执行一次。支持不同变量合并。所有`setState`合并执行后，通过引发一次组件的更新过程来引发重新绘制，之后执行回调函数（即在同步代码之后执行）。在回调函数中可访问`setState`最新值，一般用于监听组件更新是否完成。不同回调函数将按照`setState`顺序执行。【仅对存在多个setState时有效】

  > 更改`state`中数据，尽量更改为一个新的引用

- 事件处理：`react`事件自身即冒泡，冒泡到`document`上

## 指令

```jsx
// v-text
<span>{this.state.a}</span>

// v-on
// 方法中this默认不指向组件本身，为undefined
class Home extends Component {
    constructor() {
        super();
        this.state = { a: 10 };
        this.changeA = this.changeA.bind(this);// 2-早期建议，仅绑定一次
    }
    render() {
        return (
        	<div>
            	// <button onClick={this.changeA.bind(this);}>change value</button>// 1-性能问题：每次更新即调用，每次渲染产生新的函数
            	// <button onClick={this.changeA}></button>// 2-未传参，仅可获取事件对象e
            	<button onClick={e => this.changeA(e, 11)}></button>
            </div>
        );
    }
    // 2-
    // changeA(e, a) {
        // 组件变量只能在constructor中使用=赋值初始化一次，其余各处必须调用函数
        // this.setState({a});
    // }
    // 3-
    changeA = (e, a) => { 
        this.setState({a});// 11
        console.log(this.state.a);// 10
        this.setState({a}, () => {...});
    };
}

// v-for
class Home extends Component {
    constructor() {
        super();
        this.state = {
            heroes: [
                {id:1, name:'heros-1'},
                {id:2, name:'heros-2'}
            ]
        }
    }
    render() {
        return (
        	<ul>{this.state.heroes.map(item => (
                 	<li key={item.id}>{item.name}</li>
                 ))}<ul>
        );
    }
}

// v-model（浅层）
class Home extends Components {
    constructor() {
        super();
        this.state = {name: ''};
        // this.changeName = this.changeName.bind(this);
    }
    render() {
        return (
            <input type='text' value={this.state.name} onInput={e => this.setState({name:e.target.value})}/>// 原生组件可以调用，若子组件则存在性能问题
			// <input type='text' value={this.state.name} onInput={this.changeName}/>
        );
    }
    // changeName(e) { this.setState({name:e.target.value}) }
}
// v-model（深层）
class Home extends Components {
    state = { model: { data:{id:0, name: ''}, isEdit: false; } };
	// 3-
	inputHandler = e => {
        this.state.model.data.name = e.target.name;
        this.setState({model});
    };
	render() {
        return (
        	<input value={this.state.model.data.name} onInput={e => {this.setState({...this.state.model, data: {...this.state.model.data, name:e.target.value}})}} />// 1-
            <input value={this.state.model.data.name} onInput={e => {model: Object.assign(this.state.model, {data: Object.assign(this.state.model.data, {name: e.target.value})})}} />// 2-
            <input value={this.state.model.data.name} onInput={inputHandler} />// 3-可能有安全隐患
        );
    }
}

// v-if v-if+v-else v-if+v-elseif+v-else
class Home extends Components {
    constructor() {super();this.state={score:78};}
    render() {
        let level = null;
        if(this.state.score >= 90) { level=<span>优秀</span>; }
        else if(this.state.score >= 60) { level=<span>及格</span>; }
        else { level=<span>不及格</span>; }
        return (
        	// v-if
            {this.state.score >= 90 && (<span>优秀</span>)}
            // v-if v-else
            {this.state.score >= 60 ? (<span>及格</span>) : (<span>不及格</span>)}
    	   // v-if v-elseif v-else
           {level}
        );
    }
}

// v-show
class Home extends Components {
    state = {isShow: false;};// es6
    render() {
        return (
        	<div style={{display: this.state.isShow ? 'block' : 'none'}}>style属性需要对象</div>
        );
    }
}
// 动态class
class Home extends Components {
    state = {isShow:true};
    render() {
        return (
        	<div className={`a ${this.state.isShow ? 'show' : ''}`}></div>// 法1
	    	<div className={['a', this.state.isShow ? 'show' : ''].join(' ')}></div>// 法2
        );
    }
}
```

## computed

```js
// 1-不建议，性能低下。未缓存，每次render每次计算。
render() {
    let avatar = this.state.listMain.find(item => item.id === this.state.activeId).avatar;
    return (...);
}
// 2-memoize-one 减轻rerender复杂度（=>react性能优化）
// memoize-one 本质是一个高阶函数，真正的计算函数作为参数，返回一个新的函数【闭包】。新的函数内部会缓存上一次入参以及上一次返回值，如果本次入参与上一次入参相等，则返回上一次返回值，否则，重新调用真正的计算函数，并缓存入参以及结果，供下一次使用。
// yarn add memoize-one 安装依赖
import memorize from 'memorize-one';
class Category extends Component {
    getAvatar = memorize((listMain, activeId) => {
        if(activeId === 0) return '';
        return listMain.find(item => item.id === activeId).avatar;
    });
	render() {
        let avatar = this.getAvatar(this.state.listMain, this.state.activeId);
        return (
        	...
            <img src={avatar} />
        );
    }
}
// 3-useMemo hook函数组件实现computed
// 详见note-react hook.md - useMemo
```

## watch

详见`note-react hook.md - useEffect`
