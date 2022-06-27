[toc]

# 组件拆分

逻辑更加清晰，利于维护与扩展

## 容器组件

关心整个页面功能实现逻辑与数据流转。一般只对应一个jsx文件。

## 展示组件

更关注jsx渲染样式，仅关注简单的逻辑或根本不关注逻辑。一般对应jsx与样式文件。

# 组件封装（函数式调用）

函数式调用：自行调用`createElement() + render()`

```jsx
// createElement(type, [props, child1, child2...])，返回值为react element对象/react元素/虚拟dom节点。该元素是一个普通对象，完整的描述了一个dom节点的所有信息，是react应用程序的最小单位。
// jsx语法是createElement语法糖，内部编译为createElement()。
<div className='div'>
	div content
    <span>span content</span>
</div>
===>
document.createElement(
    'div', // 若需要渲染的标签是自定义标签，则此处为指定组件构造函数
    {className: 'div'}, 
    'div content', 
    React.createElement('span', {}, 'span content')
);
// 'react-dom' render(vNode, fatherDom)：虚拟dom->真实dom + 挂载到父节点，返回值为自定义组件实例。将会清空容器组件，并渲染指定dom


// Alert/Alert.jsx
// yarn add react-addons-css-transition-group 过渡动画
// 1-调用时自动创建组件
// 2-调用之前组件已经存在，调用只是显示(this)
import React, {Component} from 'react';
import ReactDOM from 'react-dom';
import ReactAddonsCssTransitionGroup from 'react-addons-css-transition-group';
class Alert extends Component {
    state = {};
	open = () => {};
	render() {
        return (
            // style class值转码，将transitionName配置为对象形式。
            // transitionEnterTimeout/transitionLeaveTimeout与styl文件中transition时长相同
            <ReactAddonsCssTransitionGroup
                transitionName={{
                    enter: style['fade-enter'],
                    enterActive: style['fade-enter-active'],
                    leave: style['fade-leave'],
                    leaveActive: style['fade-leave-active']
                }}
                transitionEnterTimeout={300}
                transitionLeaveTimeout={300}>
                {/* 内容标签不能与外层标签同时渲染，故仿v-if；内容标签需要给定key值 */}
                {this.state.isShow && (<div key='alert-container'>...</div>)}
            </ReactAddonsCssTransitionGroup>
        );
    }
}
let alertContainer = document.createElement('div');// 纯dom
document.body.appendChild(alertContainer);
let vNode = React.createElement(Alert);// 虚拟dom
let alertInstance = ReactDOM.render(vNode, alertContainer);
export default alertInstance.open;

// Alert/Alert.module.styl
.fade-enter
	opacity: 0
.fade-enter-active
	opacity: 1
	transition: all .3s
.fade-leave
	opacity: 1
.fade-leave-active
	opacity: 0
	transition: all .3s
```

## 插槽slot

```jsx
// Alert/Alert.jsx
<Dialog>
	<span>span content</span>
</Dialog>

// Dialog/Dialog.jsx
<ReactAddonsCssTransitionGroup
    transitionName={{
         enter: style['fade-enter'],
		enterActive: style['fade-enter-active'],
         leave: style['fade-leave'],
         leaveActive: style['fade-leave-active']
    }}
    transitionEnterTimeout={300}
    transitionLeaveTimeout={300}>
    {this.props.children}
</ReactAddonsCssTransitionGroup>
```

## 组合

传统组价拆分逻辑：层层渲染。渲染第一层时，在render()渲染第二层；在渲染第二层时，在render()渲染第三层，以此类推。缺点：数据逻辑散落在各个层级，没有集中管理。代码维护扩展不好。（使用context解决代代传问题仍不是最优选择。）

通过**对子组件插槽传递组件**，来实现组合技术（仅适用于中间层组件对下层组件中不传信息的场景，相当于缩短了消息传递的通道长度）。中间层组件可以通过`props.children`直接获得处理好的下层组件。若有多层组件，则数据在此传递。渲染时由父组件控制子组件，方可确定后代组件的渲染位置。

优点：可降低子组件复杂度，将业务逻辑向顶层收缩。

> 若中间层组件亦需要对下层组件传递值，则使用**`render props`**技术：祖先组件将需要传递的参数封装函数传给子组件，在子组件中调用这个函数实现参数合并，传递给后代组件。

