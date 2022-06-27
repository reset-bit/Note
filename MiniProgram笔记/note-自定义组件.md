[toc]

# 自定义组件

组件是页面的超集，页面是组件的子集。组件比页面支持更丰富的行为

```js
Component({
    // 对外公开属性
    properties: {
        count: {
            type: Number,
            value: 1,// 默认值
            observer: function() {}// watcher for properties
        }
    },
    // 组件初始数据
    data: {},
    // watcher for data & properties
    observers: {},
    // 自定义方法
    methods: {},
    // 自定义选项，如插槽
    options: {},
    // 自定义行为，如是否支持导出
    behaviors: [],
    // 导出属性与方法
    exports() {return {};}
});
```

##  组件通信

```js
/*
	1-properties + triggerEvent
	双向传值
	父组件：
		<mi-count count="{{item.count}}" bind:increase="increase" />// 父-子传递数据
		increase: function() {this.setData({...});}
	子组件：
		<button bind:lintap="increase"></button>
		properties: { count: {type:Number, value: 1} },
		methods: {
			increase: function() {this.triggerEvent('increase', null, {bubbles: true});}// 子-父发射事件，允许冒泡
		}
		
	2-selectComponent
	类似querySelector，子传父
	父组件：
		<child class="child" />
		<button bindtap="invokeChildMethod">invoke child method</button>
		invokeChildMethod: function() {
			let child = this.selectComponent(".child");
			console.log(child.num);
			child.changeNum(100);
		}
	子组件：
		Component({
			...
			behaviors: ["wx://component-export"],// 开启导出行为
			export() {
				return {// 对外暴露的属性和方法
					num: this.data.num,
					changeNum: num => {this.setData({num});}
				};
			}
		});
		
	3-其他
	如globalData\storage等
*/
```

## 计算属性computed与watcher

```js
// npm install miniprogram-computed
// 构建npm
// cart.wxml
const computedBehavior = require("miniprogram-computed").behavior;
Component({// page->component
    behaviors: [computedBehavior],
    ...
    computed: {// computed
        total(data) {...}
    },
    obervers: {// watcher
        // 首次render不执行。调用setData()即执行，不论数据是否发生变化
        // 不能修改本身依赖数据，会造成死循环
        // npm install minipragram-watch + behaviors配置可避免page->component
        a: function(newValue) {this.setData({b: newValue + 1});},// 系统调用函数，this指向组件本身
        'a, b': function(newAValue, newBValue) {...},// 监听a或b的变化
        'obj.**': function() {...}// 监听所有obj子属性的变化
    },
    methods: {...}
});
```

## 插槽

```jsx
// 默认插槽
// fa
<child><text>fa-ch message</text></child>
// ch
<slot></slot>

// 具名插槽
// fa
<child>
    <view slot="s1">fa-ch message-1</view>
    <view slot="s2">fa-ch message-2</view>
</child>
// ch
<slot name="s1"></slot>
<slot name="s2"></slot>
Component({
    ...
    options: {
        multipleSlots: true
    }
});

// 作用域插槽
// 使用抽象节点实现类似功能，father节点的表现是grandson，故grandson展示数据即达到目的
// fa 不适用条件判断
<child genereic:selectable="grandson1"></child>
// ch
<selectable num="{{num}}"></selectable>
{
    ...
    "componentGenerics": {
        "selectable": true
    }
}
// gr
<text>{{num}}</text>
Component({
    properties: {
        num: {type:Number}
    }
});
```

## 纯数据字段

在`data`中声明纯数据字段，设置时仍使用`setData()`设置，**更新不会触发整个页面`re-render`**

纯数据字段与在`this`上直接挂载属性相比更具有可读性

> `re-render`：`properties`数据变化；`setData()`传递数据给渲染层

```js
Page({
    options: {
        pureDataPattern: /^_/
    },
    data: {_child:null}
});
```

## 抽象公共数据

将多个组价间共同的逻辑代码进行封装（`vue-mixin`)，一个组件可以使用多个behaviors

```js
// commonBehaviors.js
export default Behaviors({
    data: {num: 1},
    methods: {
        printA() {console.log(this.data.a)}
    },
    lifetimes: {
        created() { console.log('created'); }
    }
});
// component1.js
import commonBehaviors from '../commonBehavoirs.js';
Component({
    behaviors: [commonBehaviors]
});
```

## 抽象节点

自定义组件的渲染内容（子组件）由调用者决定

```jsx
// fa
<child genetic:selectable="grandson1"></child>
{
    "usingComponent": {
        "child": './child/child',
        "grandson1": './grandson1/gransdon1',
        "grandson2": './grandson2/gransdon2'
    }
}
// ch
<selectable></selectable>
{
    ...
    "componentGenerics": {
        "selectable": true
    }
}
```

