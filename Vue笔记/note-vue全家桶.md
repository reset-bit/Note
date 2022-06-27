[toc]

# vue全家桶
## vue-cli

### 自动化生成文件目录解析
```js
// 修改配置文件需要重启服务器
-build// webpack配置文件
-config// webpack配置文件
	-index.js 配置代理
-node_modules// 代码文件依赖模块
-src// 源代码文件，代码分割+导入导出，浏览器无法解析。@可定位
	-router 路由相关配置
	-views 全局页面
	-components 公共组件
	-assets 公共静态资源
	-utils 工具组件
		http.js
	-router vuex组件
		index.js
-package.json// 程序配置文件
	{
		scirpt: {
			dev: webpack-dev-sever打包命令
			start: 运行dev命令
		}
		dependencies：代码中使用到的依赖包
		devDependencies: 打包使用到的依赖包
	}
```

### Runtime+Compiler和Runtime-only的区别

使用vue-cli构建项目时，有`vue build`选项，分`Runtime+Compiler`模板模式（运行时+编译器）和`Runtime-only`运行时模式（仅运行程序）。

`runtime+compiler`运行过程：`template->ast->render->virtual dom->ui`

`runtime-only`运行过程：`render->virtual dom->ui`

`runtime-only`比`runtime+compiler`更快，代码量更少，大约轻6KB，template只允许在`.vue`文件中使用，其他地方使用需要使用render函数。



## vue-router

vue核心组件。监听url变换，动态渲染指定组件

`$router`访问路由器；`$route`访问当前路由

```jsx
// router/index.js
Vue.use(VueRouter);// 在原型对象上动态开辟$router属性
const router = new VueRouter({
    linkActiveClass: 'active',// 组件激活时显示的class类名，默认为router-link-active
    routes: [
        { path: '/', redirect: '/' }
    ]
});
export default router;
// main.js or other.vue
<router-view></router-view>// 路由视图，可以通过路由指定渲染内容
<router-link to=''></router-link>// 路由切换，仍渲染成a。可使用tag指定渲染标签

// 组件缓存
// main.js
new Vue({
    router,// 通过router配置参数注入路由，让整个应用都有路由功能
    template: "<keep-alive include='list,detail' exclude=''></keep-alive>"// 仅销毁对象，还在内存中。include:要被缓存的；exclude:不被缓存的
});

// 按需加载
// router/index.js
import Cart from '../views/Cart';
const routes = [{ path: '/cart', component:Cart }];
// 改为
const routes = [{ path: '/cart', component: () => import('../views/Cart') }];
```

### 跳转传参

```jsx
// 跳转
<a v-on:click='$router.back()'></a>// 返回前一个页面
this.$router.replace();
this.$router.push();

// 传参
// 1.params传法，默认存为字符串，参数非明文显示，刷新会消失
// A.vue
<router-link v-bind:to='`/list/${item.id}`' replace></router-link>// replace替换
<router-link v-bind:to='{name:"abc", params:cid:item.id}}' replace></router-link>
<router-link v-bind:to='{path:"abc", params:cid:item.id}}' replace></router-link>
// router/index.js
{path:'/list/:cid', name:"abc"}// 动态路由参数，以:开头
{path:'/list/:cid/index'}
{path:'/list/:cid/:otherParams'}
// B.vue
this.$route.params.cid

// 2.query传法，参数明文显示，可传少量不明显数据
// A.vue
<router-link v-bind:to=`/list?cid=${item.id}` replace></router-link>
<router-link v-bind:to='{path:"/list", query:{cid: item.id}}' replace></router-link>
// router/index.js
{path:'/list'}
// B.vue
this.$route.query.cid

// 3.meta传法
// A.vue
<router-link v-bind:to='{path:"/list", meta:{cid: item.id}}' replace></router-link>
// router/index.js
{path:'/list'}
// B.vue
this.$route.meta.cid
```

### 路由守卫

路由守卫是路由跳转过程中的一些钩子函数。路由跳转是一个大的过程，这个大过程可以分为若干细小过程，每一过程都有对应的函数，可以在其中做其他操作。

```js
// 1.路由全局守卫
router.beforeEach((to, from, next) => {})// 1
router.beforeResolve((to, from, next) => {});// 导航被确认之前，所有组件内守卫和异步路由组件被解析之后 4
router.afterEach((to, from, next) => {})// 进入之后验证 4
// 2.路由独享守卫
{path: , beforeEnter: function() {}}// 2
// 3.组件内路由守卫
beforeRouteEnter((to, from, next) => {})// 不能使用this访问关键字 3
beforeRouteUpdate((to, from, next) => {})// 若在其他路由基础上跳转 3
beforeRouteLeave((to, from, next) => {})// 5
```

### 子路由

```js
// router/index.js
const routes = [
    { path: '/page1', component: Page1, children: { path: '/page1/component1', component: Component1 } }
];
```



## vuex

一个专门为vue.js应用程序开发的状态管理模式，采用**集中式存储响应式地**管理应用的所有组件状态，以相应的规则保证状态以一种可预测的方式发生变化。

构建大型单页面应用时，vuex能帮助更好的在组件外部管理状态。

> 一个足够简单的单页面应用，使用store模式完全足够所需。
>
> store（仓库）模式：定义一个store对象，对象里存储state属性即共享数据、存储操作共享数据的方法。在组件中将store.state共享数据作为data的一部分或全部，在对store.state对象里的共享数组进行改变时，调用store提供的接口进行共享数据的更改。

### vuex引入的2种方法
store：需要在data中声明过store才能访问
$store：挂载在Vue实例上的，每个组件就是一个Vue实例。可以使用this访问原型上的属性

> 通常自定义工具类如http使用`Vue.prototype.$http = http`挂载，官方组件使用`Vue.use(Vuex)`挂载
>
> `vue.prototype`与`vue.use`的区别：
>
> - 前者是在是实例上挂载属性或方法；
>
> - 后者会初始化插件对象，该对象必须具有install函数。`vue.use()`会运行导出的install函数。若插件是一个函数，则会被当做install运行

```javascript
// main.js
import store from './store';
new Vue({
    ...
    store,
    ...
});
    
// store/index.js
import Vue from 'vue';
import Vuex from 'vuex';
Vue.use(Vuex);
// 不要使用this，非指向目标对象，无意义
const store = new Vue.Store({
    // 子仓库，state默认与总仓库隔离，但getters/mutations/actions会合并，需要加上namespaced:true。
    modules: {},
        
    // 存放数据。理论上可以是任何值，实际开发中一般是对象。
    state: {},
    
    // 相当于计算属性。当多个组件在使用vuex中status数据，都要经过相同的逻辑时，建议使用getters。定义起来像函数，使用起来像数据。函数第一参数总为state。
    getters: {},
        
    // 存放方法，只有mutations的方法有权修改state中的数据，第一个参数为state。第二个参数为载荷payload，只接受两个参数。只能包含同步代码。【mutations中的方法更像是事件注册，更改state中数据的唯一方法就是提交mutation，调用mutation使用commit()】
    mutations: {},
        
    // 存放方法，没有权利修改state中的数据，可以包含异步耗时代码。若希望修改state中的数据，只能调用mutations中的方法来实现间接修改，actions内方法可以与mutations中方法同名。第一个参数总为上下文对象context（与store实例具有相同属性和方法），第二个参数同mutations。【提交的是mutation，调用action使用dispatch()】
    actions: {}
});
export default store;

// xx.vue
<xx v-text='$store.state'></xx>// state
<xx v-text='$store.getters['target']'></xx>// getters
<xx v-on:click="$store.commit('methodName', target)"></xx>// mutations
<xx v-on:click="$store.dispatch('methodName', target)"></xx>// actions

<xx v-text='$store.moduleName.state'></xx>// modules-state
<xx v-on:click="$store.commit('moduleName/methodName', target)"></xx>// modules-mutations
```

### 子仓库映射

使用辅助函数实现。注意避免映射解构出的属性或方法与state中属性或方法重名

```js
// .vue
// 1.
import {mapState, mapGetters, mapMutations, mapActions} from 'vuex';
export default {
    // mapState/mapGetters
    computed: {
        ...mapState(['key'])
        // ...mapState('address', ['key'])// 子仓库映射
        // ...mapState('other', ['otherKey'])// 其他子仓库映射
    }
    // mapMutations/mapActions
    methods: {
        ...mapActions({'changeKeyPlus':'changeKey'})
        // ...mapActions('address', {'changeKeyPlus':'changeKey'})// 子仓库映射
    }
};
// 2.不支持一对多映射仓库
import {createNamespacedHelpers} from 'vuex';
const {mapState, mapGetters, mapMutations, mapActions} = createNamespaceHelpers('address');// 解构
export default {
    computed: {
        ...mapState(['key'])
    },
    methods: {
        ...mapMutations(['changeKey'])
    }
};

// html
<xx v-text='key'></xx>
<xx v-on:click='changeKey(101)'></xx>
```



## axios

axios是一个基于Promise的、用于浏览器和node.js的http客户端，本质上是对XHR对象的封装，只是Promise的实现版本。本身可以：

浏览器创建XHR对象；客户端防CSRF；提供一些并发接口；拦截请求和响应；自动转换JSON数据等。

```javascript
// 封装axios: utils/http.js
import axios from 'axios';

function http(userOpions) {
    let defaultOptions = {method:'get'};
    let ajaxOptios = Object.assign({}, defaultOptions, userOptions);// 伪深拷贝，需要再合并一层headers
    ajaxOptions.headers = Object.assign({}, {Authorization: sesssionStorage.getItem('token'), userOptions.headers});
    return axios('ajaxOption')
    		.then(res => {
                if(res.status === 200) {
					switch(res.data.code) {
                        case 200:
                            return res.data.data;
                            break;
                        case 199:
                        case 401:
                        case 404:
                        case 500:
                            throw new Error(res.data.msg);
                       }
                } else {
                    throw new Error(res.statusText);
                }
    		})
    		.catch(err => {
        		console.log(err.message);
        		return Promise.reject();// 返回一个失败的promise对象：默认catch总返回一个成功的promise对象，会进入封装后外调的then()。为了避免出错情况仍然进入成功的回调函数，需要返回一个失败的promise对象
		    });
};
export default http;

// 导入 main.js
import http from './utils/http.js';
Vue.prototype.$http = http;// 挂载到原型对象上
```

