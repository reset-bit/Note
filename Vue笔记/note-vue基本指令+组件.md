[toc]

# Vue基本指令

```js
// 声明式渲染
// 基本指令
v-text // 控制双标记中显示的内容
v-for (v-bind:key) // 迭代渲染数组类型的数据；对渲染的数据作唯一标识，最好使用数据中唯一id
v-bind:xx / :xx // 标签属性动态赋值
v-on:xx / @:xx // 事件绑定，参数为函数地址（函数名），单行操作可以直接写
// 频繁切换使用v-show，否则使用v-if
v-show // 控制元素可见性，绑定值为boolean。控制台标签仍存在。具有更高的初次渲染成本，更低的切换成本
v-if v-else-if v-else // 控制元素是否存在。控制台标签可能不存在。具有更低的初次渲染成本，更高的切换成本
v-model="" // 表单元素双向数据绑定
v-slot// template标签+具名插槽使用

// 事件修饰符，写在标签上
@click.prevent// 阻止事件默认行为
@click.stop// 阻止事件冒泡
@click.captrue// 绑定事件到捕获阶段
@click.once// 事件绑定函数只执行一次
@click.self// 只关注自身事件，不关注冒泡
@click.left// 点击鼠标左键触发
@click.right
@click.middle
v-model.tirm// 去除表单元素空格
v-model.number// 将绑定的表单数据转换为数字
v-model.lazy// 监听change而不是input
```

> 1. `v-if`与`v-for`不能同时使用：`v-for`指令优先级高于`v-if`，当二者处于同一节点时，`v-if`将分别运行于每个`v-for`循环子项中；
>
> 2. `v-for`中`key`的作用：在虚拟dom算法中，在新旧node对比时辨识VNode。即**基于key的变化重新排列元素顺序，会移除key不存在的元素。**如果不同key，vue会使用一种最大限度减少动态元素并尽可能的尝试**就地修改**/复用相同类元素的算法；
>
> 3. `v-model`是`v-bind+@input`语法糖：
>
> 	```html
> 	<ChildComponent v-model="pageTitle" />
> 	<!-- 是以下的简写: -->
> 	<ChildComponent :value="pageTitle" @input="pageTitle = $event" />
> 	```
>
> 4. $默认为vue实例对象，也就是vue中的this
>
>    ```js
>    // 常用$属性
>    $refs// 标识dom对象
>    $parent $children// 标识父组件、子组件
>    $router// 访问路由器，包括常用方法push/go/replace等
>    $route// 访问路由，包括meta/query/params等属性
>    ```
>

## MVVM简述

把变量看作Model，把HTML某些DOM节点看作View，并假定它们之间被关联起来了。我们并不操作DOM，而是直接修改JavaScript对象。改变JavaScript对象的状态，会导致DOM结构作出对应的变化。

```js
// main.js
// vm实际是viewModel视图模型，vue数据必须通过vm操纵视图。即：
let vm = new Vue({
    // 绑定的dom元素，参考index.html中div结点id值
	el: '',
    // 将指定template加载到<div id="app"></div>挂载点上
	template: '',
    // 页面中要渲染显示的数据、完成功能需要的辅助数据
	data: {},
	// 数据处理函数，this指向vue对象，访问data数据需要加this
	methods: {},
    // 注册组件
    components: {}
});

vm.$el// 获取vue实例关联的dom元素
vm.$data// 获取vue实例的data选项
vm.$options// 获取vue实例的自定义属性，如vm.$options.methods
vm.$refs// 获取页面中所有包含ref属性的dom元素，如果有多个元素，返回最后一个。如vm.$refs.hello + <xx ref='xxx'></xx>
// vm.$data.aa <==> vm.aa
xxx.$emit('') | xxx.$on('', function() {}) | xxx.$off('')// 向父组件发送事件
$mount()// 拿到html模板作为template，然后将这个template通过compileToFunctions()编译成render函数

<style scoped></style>// scoped能够实现组件样式私有化，不会对全局造成样式污染。原理是对scoped组件添加一个标识唯一的字符属性。
```

## 数据驱动简述

通过改变数据来完成页面内容的改变。每一个指令都会有一个对应的用来观测数据的对象，叫做`watcher`。如`v-text='message'`即为一个`watcher`。`watcher`对象中包含了待渲染的关联dom元素。

vue中存在3种`watcher`：

1. 数据`render watcher`，数据改变时视图需要改变
2. `computed watcher`是`computed`函数在自身内部维护的`watcher`，配合内部属性判断`computed`的值是重新计算还是直接复用以前的值
3. `watcher api`，即用户自定义导出`watch`


# 组件化开发

SPA单页面应用程序（Vue组件化开发，页面即组件，多页面切换即组件切换）；MPA多页面应用程序

组件：封装的一段html代码，本质是一个**拥有预定义选项的Vue实例**
定义组件：主要解决什么问题、不解决什么问题，保持好的复用性、解耦合性

## 全局组件与局部组件

`vue 2.x`，以基本的`.html`文件为例

```jsx
<div id="app">
	<cpn></cpn>// 3.使用
    <cpn1></cpn1>
</div>
<script src='../vue.js'></script>
<script>
    // 1.创建组件构造器
	const cpnC = Vue.extend({
        template: '<div>this is a div</div>'
    });
    const cpnS = Vue.extend({
        template: '<div>this is a div, too</div>'
    });
    // 2.注册组件
    Vue.component('cpn', cpnC);// 全局组件
    const app = new Vue({
        el: '#app',
        // 局部组件（多）
        components: {
            cpn1: cpnS
        }
    });
</script>
```

## 事件绑定与核心选项
```html
<!-- .vue文件 只包括一个根节点 -->
<template>
    <button v-on:click='fun1'>fun1</button>
    <button v-on:click='fun2(100)'>fun2</button>
    <button v-on:click='fun3($event, 200)'>fun3</button>
</template>

<script>
    export default {
        name: 'MiCount',// 缓存时要用name缓存，调试时便于调试
        data() {return {};}// 返回的对象中存放组件所需数据，Vue对象中为data:{}

        // -----------------lifecycle hooks生命周期钩子函数-----------------
        【在beforeCreate之前：初始化事件和生命周期】
        // -----------------创建：在内存中创建
        beforeCreate() {}// 在实例初始化之后，不可以获取data/methods，可以访问路由及其数据：this.$router
        created() {}// 在实例创建完成后被立即调用，完成数据观测 (data observer) 、属性与方法的计算、watch/event事件回调，$el不可见（不能进行dom操作）。可以获取data/methods。
    	【ajax初始化数据建议在此处。因为在beforeCreate()中，无法获取data对象。尽管ajax异步，但该函数也无法获取methods中函数，无法调用方法。另外，dom初始化渲染和请求初始化数据没有什么冲突】
        【判断实例化的vue对象中有无el选项，没有就使用vm.$mount()手动挂载模板】
        // -----------------挂载：渲染到dom
        beforeMount() {}// render函数首次被调用，特点与created相似
        mounted() {}// 已完成模板或el对应html渲染后（挂载完毕）调用。可以进行dom操作
        // -----------------更新：更新已使用data变量、路由
        beforeUpdate() {}// 在更新前根据需要修改data数据，本次更新生效，不会再触发更新
        updated() {}// 进行dom相关操作，动态渲染ajax请求数据建议此处
        // -----------------激活
        activated() {}// 组件被keep-alive缓存后，从无到有、从不激活到激活调用activated
        deactivated() {}// 组件被keep-alive缓存后，从激活到不激活调用deactivated
        // -----------------销毁
        beforeDestroy() {}// 释放组件占用的资源和内存空间，如清除计时器、销毁卸载watcher
        destroyed() {}
        // -----------------lifecycle hooks生命周期钩子函数-----------------

        computed: {data1(){}}// 计算属性，定义起来像函数，必须显式return，使用时当做data数据使用。使用到的数据称为依赖数据。计算属性默认只能用，不能直接赋值改。通过改变依赖数据间接修改计算属性的值。默认是getter，可显式设置setter，即computed: {data1:{get:function(){},set:function(){}}}
        watch: {}// watch可以监听任何想监听的数据，不限于data/computed。比较消耗性能，能不用就不用
        methods: {
            // 事件绑定:若参数为函数地址（名），则该函数就是和事件绑定的函数；否则，vue将在参数外封装一层匿名函数，并携带事件参数$event作为形参
            fun1: function(e) { console.log(e); },// 不能传自定义参数
            fun2: function(num) { console.log(num); },// 不能传事件对象e
            fun3: function(e, num) {// 写法稍复杂
                console.log(e);
                console.log(num);
            }
        }
    	// ----------组件传值节点------------
        // 子组件中设置，接收外部参数，相当对外暴露的公有数据，不能和data、computed内属性重名；命名建议使用maxCount。局限：不能指定是否必传、数据类型等
    	// props: ['count']
        props: {
            'count': {
                required: true,
                type: Number
            }
        }
        model: {
            prop: ''// 指明props中双向数据关联节点prop
            event: ''// $emit发送事件名称
        }
	    // provide可写成对象/函数，对象只能传常量；建议写成函数返回一个对象
    	// provide: {data:'data'}
        provide() {return {data:this.data};}
        inject: ['a']
    };
</script>
```
1. `created`和`mounted`区别：
   - `created`在模板渲染成html前调用，即通常初始化某些属性，之后再渲染成视图；不能进行dom操作。
   - `mounted`在模板渲染成html后调用，通常是初始化页面完成后对dom节点进行需要的操作、发送ajax请求页面需要的初始数据。可以进行dom操作。
2. `computed`和`watch`区别：
	- 计算属性是【关注一个或多个数据的变化，得到一个最新的结果】，基于响应式依赖进行缓存（如果不希望有缓存，可用方法来替代）。
	- watch用于【监听某一数据的变化，进行一系列的操作】。


# 组件拆分

## 父子组件传值（含祖先与后代）

- 传值时注意数据类型，v-bind可改变数据类型
- 对子组件添加class值，将添加到子组件的根节点上

```javascript
/**	
	!!单向数据流，子组件只能使用父组件的数据，不能修改。
	1.props
	子组件props规定对外暴露的接口数据，由父组件进行数据或方法的传递（html命名方式建议maxCount->max-count）。
	缺：非父子关系的后代关系需要代代传；可能需要传递很多参数，耦合性强。
	eg: 
		子组件：props:['count']
		父组件：<MiCount v-bind:count='1' />
	
	2.props + $emit
	子组件使用$emit触发父组件自定义事件（可选传值），而不关心父组件需要处理的业务逻辑。父组件使用v-on:xxx/$on('xxx', function() {})进行事件监听并进行响应。
	优：子组件仅产生事件通知父组件，无需关心父组件如何应对，极大降低耦合性。
	eg:
		子组件：<button v-on:click='$emit('decrease')'>-</button>
		父组件：<MiCount v-on:decrease='count -= 1;' />
		
	3.props + $emit + v-model + model节点
	父组件使用v-model传递交付数据，子组件使用model节点接收，进行双向数据绑定。【自定义组件v-model传值】
	优：对多处简单值修改友好，如MiCount-decrease/MiCount-increase/MiCount-input。
	缺：仅适用简单值修改逻辑。
	eg:
		子组件：model:{prop:'count', event:'change'}
			  <button v-on:click='$emit('change', count - 1)'>-</button>
		父组件：<MiCount v-model='count' />
	
	4.$children/$parent
	父组件用$children找到所有子组件后进行过滤（在mounted()及其之后生命周期函数中调用），直接对子组件属性和方法进行调用赋值。子组件对父组件$parent亦然。
	缺：耦合过高，不易管理。
	eg:
		父组件：data() {msg:''}
			   methods: {printMsg() {}}
		子组件：<button v-on:click='$parent.printMsg()'></button>
			   <button v-on:click='$parent.msg='hello'></button>
	
	5.provide/inject（祖先和后代组件传值）
	父组件通过provide节点规定给子组件传递的值，子组件通过inject节点接收值。往往在自定义层次深的组件时使用
	优：离的很远的祖先和后代组件可以互相传值。
	缺：不是可响应的。如果传递的数据是对象，【对象属性】是可响应的。
	eg:
		祖先组件：provide: {data:'10086'} / provide() {return {data: this.data, changeData: this.changeData};}
		后代组件：inject: ['data', 'changeData']
				<span v-text='data'></span>
				<button v-on:click='changeData('95588')'></button>
	
*/ 
```

## 任意组件传值

```javascript
/**
	1.eventBus
	直接挂载事件总成对象eventBus，任何组件都可以拿到事件总成对象的$emit、$on。
	优：任意组件间都可以通过事件对象进行数据传递。
	缺：$emit、$on过多；在created中$on，在beforeDestroy中$off。
	eg:
		main.js: Vue.prototype.$eventBus = new Vue();
		组件：created() {this.$eventBus.$on('xxx', function(data) {})}
		     beforeDestroy() {this.$eventBus.$off('xxx')}
			 <button v-on:click='$eventBus.$emit('xxx', data)'></button>
			 
	2.vuex
	优：所有组件可数据传递。
	缺：不是所有数据都适合放在vuex中。
	
*/
```

## 自封装函数式组件（slot）

```jsx
// 在MiNotice/index.js中导入MiNotice.vue，注册dom并挂载
// 在MiNotice/MiNotice.vue mounted()定时销毁

// slot: 公共组件template中使用slot占位
// 1.基本插槽
// MiDiaglog/index.vue
<slot></slot>
// MiAlert.vue
<MiDialog></MiDi>

// 2.具名插槽：在指定位置插入html。不带name的slot出口会带有一个隐含的名字default
// MiDialog/index.vue
<slot name='a'></slot>
// MiAlert.vue
<img slot='a' />
<template v-slot:a></template>
    
// 3.作用域插槽：父组件希望使用子组件的数据，可在slot-scope中指明渲染依赖的数据（数据可能与传递的初始数据不同，是指具体渲染的每条数据，类似v-for。与关联的要求数据有关，如element-ui el-tree中为树形数组单条数据）。延伸了子组件的作用域。
// mi-dialog/html
<slot :num='1'></slot>
// mi-alert/html
<MiDialog>
    <template slot-scope='{num}'>
        <span v-text='slotObjnum'></span>
    </template>
</MiDialog>
```

## hook

在一个生命周期中监听其他生命周期，一些情况下可以简化和优化代码

```js
// .vue
// 使用定时器
export default {
    mounted() {
        this.timer = setInterval(() => { ... }, 1000);
    },
    beforeDestory() { clearInterval(this.timer); }
}
// hook
export default {
    mounted() {
        const timer = setInterval(() => { ... }, 1000);
        this.$once('hook:beforeDestory', () => { clearInterval(timer); });
    }
}
// 父组件监听子组件生命周期
<v-chat @hook:mounted='loading = false'
        @hook:beforeUpdated='loading = true'
        @hook:updated='loading = false'
        :data='data'/>
```

