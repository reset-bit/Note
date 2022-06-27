[toc]

# react-router-dom(->vue router)

## v5->v6
v6相比于v5带来了破坏性的API，同时建议使用hook

1. 规范组件
    ```jsx
    <Router />// 基础路由，可以嵌套
    <Routes />// 代替Switch。通常一个应用只有一个Routes组件，但可以根据实际需要定义多个路由出口
    <Route path='/' element={<Pages.Home/>}/>// 可以嵌套
    <Link />// 导航组件
    <Outlet />// 自适应渲染组件
    ```
2. 渲染模式
v5中提供`component、render、children`三种方式渲染组件（见组件渲染）；v6统一规范为`element`，使用`useParams()/useLocation() hook`获取参数
3. 排序方法
v5路由算法为手动排序；v6会计算返回优先级高的路由组件
4. 组件子路由
v5嵌套路由需要（使用`hook useRouteMatch()`获取`match`对象）拼接绝对路径来渲染组件子路由；v6在使用`<Link />或<Route />`时只需要定义父路径的相对路径即可，v6内部会自动拼全路径
5. v6带来`Outlet`组件，用于渲染当前路由下的子组件

## 路由跳转

### 基础配置
```jsx
// App.jsx
import { Switch, Route, Redirect } from 'react-router-dom';
import Pages from './pages';
class App extends React.Component {
    render() {
        return (
        	<Switch>
            	// 均为模糊匹配，使用exact精确匹配
            	<Route path='/' exact><Redirect to='/home' /></Route>
            	// 始终优先匹配能匹配到的
            	<Route path='/home' component={Pages.Home}></Route>
            </Switch>
        );
    }
}
// index.js
// 根实例注入HashRouter
import { HashRouter } from 'react-router-dom';
reactDom.render(
	<HashRouter><App/></HashRouter>,
    document.getElementById('root');
);
```

### 点击跳转

```jsx
import { Link } from 'react-router-dom';
export default () => {
    return (
        <Link to='/introduce'>点击跳转</Link>
    );
};
```

### 事件跳转

```jsx
// v5 useHistory hook
import {useHistory} from 'react-router-dom';
export default function Detail() {
    const history = useHistory();
    const goHome = () => {
        history.push('/home');
        // history.replace('/home');
    };
}

// v6 useNavigate hook
import {useNavigate} from 'react-router-dom';
export default function Detail() {
    const navigate = useNavigate();
    const goHome = () => {
        navigate('/', { state: {id: 123} });
        // navigate('/home', {replace: true});
    };
}
```

### 代理跳转

```jsx
<Link to='/home' replace></Link>
history.push('/home');
// cart -> login -> order_confirm
// cart
history.push({pathname:'/order_confirm', query:{id:10}});
history.push({pathname:'/login', state:{from:location}});// ...登录判断逻辑
// login
history.replace(location.state.from || '/home')
// 充当路由守卫（登录验证：路由跳转+401劫持）
class App extends Component {
    render() {
        return (
            <Switch>
            	<Route path='/' exact><Redirect to='/home'/></Route>
                // v5 3种渲染组件方式
            	<Route path='/home' component={Home}/>
                <Route render={({history, location, match}) => {
                        return sessionStorage.getItem('token') ? (
                        	<Switch>
                            	<Route path='\address' component={Address}>
                            </Switch>
                        ) : (
                        	<Redirect to={{pathname:'/login', state:{from: location}}}></Redirect>
                        );
                    }}/>
                <Route children={() => {...}}/>
            </Switch>
        );
    }
}
// 401
// yarn add history
```

## 路由分发

### 一级路由

```jsx
// App.jsx
class App extends Component {
    render() {
        return (
            // v6版本中Switch已移除，使用Routes代替
            <Switch>
                // v6版本中Redirect已移除，使用Navigate代替
            	<Route path='/' exact><Redirect to='/home'/></Route>
            	<Route path='/home' component={Home}/>// 1-
                <Route path='/home' children={<Home/>} />// 2-
                <Route><Home/></Route>// 3-
            </Switch>
        );
    }
}

```

### 二级路由

```jsx
// Login/index.jsx
class Login extends Component {
    render() {
        return (
        	<Switch>
            	<Route path='/login' exact><Redirect to='/login/pwd' /></Route>
            	<Route path='/login/pwd' component={LoginPwd}/>
            </Switch>
        );
    };
}
```

### 懒加载

```jsx
import React from 'react';
import { Routes, Route } from 'react-router-dom';
const Detail = React.lazy(() => import('./pages/Detail/Detail.jsx'));
const App = () => {
        return (
                <Routes>
                        <Route path='detail/:target' element={ <React.Suspense fallback={<>...</>}><Detail/></React.Suspense> }/>
                </Routes>
        );
};
```

## 常用路由模式

```jsx
// index.jsx-Router
// 低阶Router，条件命中时不会跳出条件判断函数的外围。
import React from 'react';
import reactDom from 'react-dom';
import { Router } from 'react-router-dom';
import App from './App.jsx';
import history from './router/history.js';// 成功待定
reactDom.render(
	<Router history={history}><App /></Router>,
    document.getElementById('root')
);
// router/history.js
import {createBrowserHistory} from 'history';
export default createBrowserHistory();

// index.jsx-HashRouter
// 使用hash模式同步组件与URL。使用静态文件服务器，建议使用HashRouter
import React from 'react';
import reactDom from 'react-dom';
import { HashRouter } from 'react-router-dom';
import App from './App.jsx';
reactDom.render(
	<HashRouter><App /></HashRouter>,
    document.getElementById('root')
);

// index.jsx-BrowserRouter
// 使用HTML5的history API，让页面UI和URL同步。有服务器需要响应web的请求，建议使用BrowserRouter
import React from 'react';
import reactDom from 'react-dom';
import { BrowserRouter } from 'react-router-dom';
import App from './App.jsx';
reactDom.render(
	<BrowserRouter><App /></BrowserRouter>,
    document.getElementById('root')
);
```

## 路由传参

```jsx
// v5
// 路由对象
value: {
	// history与location被react-router-dom放置在被路由直接渲染的组件props中
    // 支持一系列方法如push()/replace()/back()
    history:[
        {
            ...
            location: {
                ...
                pathname: '/address',// vue为path
                state: undefined,
                query: ''
            },
        }
    ]
    // 包含了一个路由的完整信息。location = history[history.length - 1].location
    location: { ... pathname: '/address', state: undefined, query: '' },
    // 对location.pathname中参数的正则表达式提取+封装
    match:{ params:{} }
}

/*
    1-state
        优：加密藏值，支持复杂对象
        缺：只有browserHistory模式支持state藏值
        http://xxx/密文
        放：history.push({pathname:'/home', state:{id:10}});
        取：location.state.id

    2-query
        仅支持少量、不重要、无需加密数据
        http://xxx?id=10&name=zhangsan
        放：history.push({pathname:'/home', query:{id:10}});
        取：location.query.id

    3-match/params
        比query优雅，自动转为字符串。仅支持少量、不重要、无需加密数据
        http://xxx/10/zhangsan
        放：<Link path={`/home/${id}`}/>
        App.jsx：<Route path='/list/:id' component={Pages.List}/>
        取：props.match.params.id
*/

// v6 
// 刷新页面参数不丢失
// 1-路由形式携带 params
// 缺：只能传字符串，需要在路由中配置
// App.jsx
<Route path='/list/:id' element={<List />}></Route>
// List.jsx
import {useParams} from 'react-router-dom';
export default function List() {
    const {id} = useParams();
}
// 2-以url参数形式携带 searchParams
// 优：不需要在路由中配置
// 缺：只能传字符串，参数在url中以明文显示
// List.jsx
<Link to={'/detail?id=123'}></Link>
// Details.jsx
import {useSearchParams, useLocation} from 'react-router-dom';
import qs from 'query-string';// 使用useLocation解析结果为url字符串
export default function Detail() {
    // 1
    const [searchParams, setSearchParams] = useSearchParams();
    let id = searchParams.get('id');
    // 2
    const {searchParams} = useLocation();// ?id=123
    let id = qs.parse(searchParams);// {id: 123}
}
// 3-state
// 优：可传递对象，不会在url中明文显示
// List.jsx
<Link to={'/detail'} state={{id: 123}}></Link>
// Detail.jsx
import {useLocation} from 'react-router-dom';
export default function Detail() {
    const {state} = useLocation();
    let id = state.id;
}
// 4-useNavigate hook跳转传参
// 见跳转传参

// 另 hook：useLocation/useHistory
```
