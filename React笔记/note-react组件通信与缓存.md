[toc]

# 组件通信

## 父子组件（含祖先与后代）

单向数据流！子组件可调用父组件传来的函数，实现间接修改父组件值。

```js
/*
1.props
子组件自带props属性，用以获取外部组件对自身的传值。支持批量传值、传值限定。
eg:
	子组件：<span>{this.props.val}</span>
			<span>{this.props.key1}</span>
	父组件：<Child val={this.state.valueToChild} />
			<Child {...this.state.obj} /> ===> <Child key1={this.state.obj.key1} key2={this.state.obj.key2}>
	// yarn add prop-types安装依赖以限定数据
	子组件：class Child extends Component {
				// 1-es7
				static defaultProps = {b:true};
				static propTypes = {
					b:PropsTypes.bool, 
					a:PropsTypes.number.isRequired,
					obj:PropTypes.shape({
						key: PropTypes.string.isRequired
					}).isRequired
				};
				render() {}
			}
			// 2-其它
			Child.defaultProps = {};
			Child.propTypes = {};
	// 传递函数以改变父组件值。父组件传递时不建议将函数体写到标签内
	子组件：<button onClick={() => this.props.changeValue(101)}></button>
	父组件：<Child val={this.state.valueToChild} changeValue={this.changeValue}/>
	
1.5.代代传（不建议）

2.context（祖先和后代组件传值）
另建文件创建上下文对象并导出，在数据提供方【祖先】放入需要传递的值（多个值整合为对象），在数据接收方【后代】中使用context获取（需要存在层级关系）。
缺：消耗内存，不建议过度使用
eg:
	// categoryContext.js
	import { createContext } from 'react';
	export default createContext({a:200});// 初始值，后续再提供会替换
	// 祖先组件
	import context from './categoryContext.js'
	class Father extends Component {
		render() {
			return (
				<context.Provider value={{b:199}}>
					<Child />
				</context.Provider>
			);
		}
	}
	// 后代组件
	// 类组件
	import context from './context.js';
	class GrandSon extends Component {
		static contextType = context;// 1-
		render() {
			return (
				<span>{this.context.b}</span>
			);
		}
	}
	GrandSon.contextType = context;// 2-
	// 函数组件
	import context from './context.js';
	export default () => (
		<context.Consumer>
			{({b}) => (
				<span>{b}</span>
			)}
		</context.Consumer>
	);
	
3.useRef（子传父）
参见note-react hook.md
*/
```

## 任意组件

```js
/*
1.eventBus
yarn add events安装依赖包，另建文件创建事件发射器对象并导出。先监听再发射。
优：支持任意组件间传值
缺：需要避免过度使用（到处emit）
// eventEmitter.js
import { EventEmitter } from 'events';
export default new EventEmitter();
// child.jsx--发射事件
import emitter from './eventEmitter.js';
class Child extends Components {
	render() {
		return (<button onClick={() => emit('hello', 'message')}>emit event</button>);
	}
}
// child2.jsx--监听事件
import emitter from './eventEmitter.js';
class Child2 extends Components {
	componentDidMount() {// 注册
		this.helloListener = emitter.addListenser('hello', data => {
			console.log('received message: ' + data);
		});
	}
	compomemtWillUnmount() {// 销毁
		emitter.removeListenser(this.helloListener);
	}
}

2.redux
缺：适合大型项目使用，学习成本较高(类vuex-vue)
*/
```

# 组件缓存

react函数式组件渲染页面及交互的本质是，让开发者避免使用dom操作API，使用jsx语法就可以创建dom元素。jsx语法需要经过babel jsx语法转义为React.createElement（详见note-react组件拆分与封装），之后交由ReactDOM.render渲染。

react**类组件**会继承PureComponent，对传入props进行浅比较。**函数组件**则每次重新执行函数，内部产生新的引用地址。

对函数组件而言，只要dom数据结构引用地址及内容没变，组件就会缓存。所以**组件渲染缓存优化可以遵循**：把渲染性能开销大的组件提升父级、使用父级props传递组件。

## 优化子组件更新

没有必要更新子组件：父组件传值没有改变、没有使用父组件传值等

```jsx
// 1-基础版
// 父组件仅影响子组件props。若子组件没有state属性则state=null，称为无状态组件。
class Child extends Component {
    /**
     *	@return true 需要更新
    */
    shouldComponentUpdate(nextProps, nextState) {
        let keys = Object.keys(nextProps);
        for(let i = 0; i < keys.length; ++i) {
            if(this.props[keys[i]] !== nextProps[key[i]]) return true;
        }
        // 避免子组件自身无法更新
        if(this.state && nextState) {
            keys = Object.keys(nextState);
            for(let i = 0; i < keys.length; ++i) {
                if(this.state[keys[i]] !== nextState[keys[i]]) return true;
            }
        }
        return false;
    }
}

// 2-简化伪平替
// 对无状态组件：可继承自Component子类PureComponent，该类内部实现shouldComponentUpdate()。本方法不调用shouldComponentUpdate()，仅到componentWillReceiveProps()为止。显式书写shouldComponentUpdate()将覆盖。
// 浅比较(shadow compare)：基本类型可靠。
class Child extends PureComponent {}
// 注意除了传递对象等引用类型数据的情况之外，将函数体直接写在子标签内同样会使子组件更新：父组件更新时执行render()，该函数每次将产生一个相同的新函数给子标签。由于函数是引用类型，且地址发生了变化，PureComponent认为该子标签props发生了变化，因此选择更新。【本限制仅对父子组件传递有效】
class Father extends Component() {
    state = {
        a: 10,
        obj: { key: 100 }
    };
	changeA = (val) => {this.setState({a:val})};// 防止子组件更新
	changeObj = (val) => {// 令子组件更新
        this.state.obj.key = val;
        this.setState({obj: {...this.state.obj}});// 浅拷贝，同Object.assign()
    };
	render() {
        return (
            <button onClick={() => this.changeObj(101)}>changeObj</button>
        	<Child changeA={() => this.changeA(11)} />
        );
    };
}
```

