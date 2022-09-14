[toc]

# Angular基础

优点：模块化；数据双向绑定；MVC模式；依赖注入；指令系统

缺点：对IE6/7兼容不好；最佳实践教程少；复杂与代码膨胀

## 数据绑定

Angular仅对包含特殊声明的元素进行处理，如指令、值绑定、过滤器、表单控件等

```jsx
<html ng-app>// 声明所有包含的元素属于Angular应用
	<head>...</head>
    <body>
        <div ng-controller='MyController'>// 声明所有包含的元素属于控制器
        	<input ng-model='name' type='text'/>
	        <h1>hello {{clock.now}}</h1>// 在视图中通过对象的属性而非对象本身来进行绑定是Angular最佳实践
        </div>
        <script>
        	function MyController($scope, $timeout) {
                function updateClock() {
                    $scope.clock = new Date();
	                $timeout(() => {updateClock();}, 1000);
                }
                updateClock();
            }
        </script>
    </body>
</html>
```

## 模块-module

优点：保持全局命名空间清洁；编写测试代码更容易；易于在不同应用间复用代码；令应用以任意顺序加载代码的各个部分

```js
// 声明模块
angular.module('myApp', []);// 模块名称 依赖列表
// 获取应用
angular.module('myApp');
```
## 控制器-controller

作用：增强视图，向视图中添加额外的功能；不适合DOM、数据操作；设计良好的应用将复杂逻辑放到指令和服务中

```js
angular.module('myApp', [])
	// $scope 与指令相关的作用域
	// $element 当前指令对应的元素
	// $attrs 当前元素的属性组成的对象
	// $transclude 嵌入链接函数，实际被执行用来克隆元素和操作DOM
	.controller('MyController', ($scope, $element, $attrs, $transclude) => {
        $scope.onLogin = foo();// 内容被指令控制
    })
```


## 作用域-scope

作用域对象是应用业务逻辑、控制器方法、视图属性的地方。它充当**数据模型**，是视图和控制器之间的桥梁

每个angular应用有且仅有一个root scope。指令会创建子级作用域，它会嵌入在父级作用域中，构成作用域分层结构
scope是一个普逼的js对象，其拥有的部分属性允许导航（如\$root、\$parent）、发布与订阅事件（如$on、\$emit、\$broadcast)等

**作用：**提供`$watch`来监听数据模型的变化，将该变化通知整个应用；隔离业务功能和数据；提供表达式运算执行环境等

**生命周期：**

- **创建**：创建控制器或指令时，Angular使用`$injector`创建一个新的作用域。并在该控制器或指令运行时将作用域传递进去。
- **链接**：Angular启动并生成视图时，将作用域附加或者链接到视图中，注册上下文发生变化时运行的函数`$watch`。根元素ng-app和`$rootScope`进行绑定，后者是所有$scope对象的最上层。
- **更新**：事件循环通常执行在$rootScope上，每个子作用域执行自己的脏值检测。若检测到变化则触发指定的回调函数。
- **销毁**：当一个$scope在视图中不再需要时，该作用域将会自己清理自己

## 表达式-{{}}

在归属所有域中执行；不会抛出`TypeError、ReferenceError`异常；不允许使用流程控制语句；接收过滤器与过滤器链

Angular在`$digest`循环中自动解析表达式，使用`$parse`内部服务进行表达式运算

```js
// 手动解析表达式
angular.module('myApp', [])
	.controller('MyController', ($scope, $parse) => {
        // 视图：<input ng-model='expr'/> <h2>{{parseValue}}</h2>
        $scope.$watch('expr', (newVal, oldVal, scope) => {
            if(newVal !== oldVal) {
                let parseFun = $parse(newVal);
                $scope.parseValue = parseFun(scope);
            }
        });
    })
```

## 过滤器

格式化需要展示给用户的数据

```js
{{ 123.456 | number:2 }}

// 内置过滤器
currency// 转换成货币
date// {{ today | date:'medium' }}
filter// 从给定数组中选择子集返回，参数可传字符串、对象、函数、true/false
json// 将object转换为json
limitTo// 字符串/数组截取，支持负数
lowercase/uppercase
number// 将数字格式化为文本
orderBy// 参数可传函数、字符串、数组元素

// 自定义过滤器：将字符串第一个字母转为大写
angualr.module('myapp.filters', [])
.filter('capitalize', input => {
    input && return input[0].toUpperCase() + input.slice(1);
});
```

### 表单验证

Angular在处理表单时，会根据表单当前状态添加一些css类，如`ng-pristine、ng-valid`等。

```html
// 内置验证指令
<input type='text' required/>
<input type='text' ng-minlength='5'/>
<input type='text' ng-maxlength='20'/>
<input type='text' ng-pattern='[a-zA-Z]'/>
// 数据绑定
<input type='email' ng-model='user.email'/>
// 数据获取
formName.inputFileName.property
formName.inputFileName.$printine// 判断用户是否修改了表单，修改为true
formName.inputFileName.$dirty// 只要修改过表单值，该值为true
formName.inputFileName.$valid// 表单属性合法，该值为true
formName.inputFileName.$invalid
formName.inputFileName.$error// 该对象包含当前表单的所有验证内容，以及是否合法的信息
```

## 生命周期

### 编译

遍历HTML文档，根据文档中的指令定义来处理页面上的指令。

模板树可能又大又深，编译后的模板会返回一个模板函数。我们能够有机会**在指令的模板函数被返回前，对编译后的DOM树进行修改**。这时DOM树还没有进行数据绑定，此时对DOM树进行操作只有很小的性能开销，ng-repeat、ng-transclude等内置指令即在此时对DOM进行操作。

### 链接

设置事件监听器，监视数据变换并实时操作DOM。

## 依赖注入与控制反转

依赖注入是一种编程技巧，控制反转是一种编程思想，即**控制反转是依赖注入的一种代码设计思路**。

控制是对程序流程的控制，反转是将实例化的控制权从当前程序手中反转到了外层框架。最终是为了解决数据共享和代码复用的问题。

**优点**：松耦合，易维护扩展

**缺点**：复杂与抽象，编译时错误被推到运行时

```js
// 非依赖注入
class NotificationComponent {
    msg: MessageService;
    constructor() {
        this.msg = new MessageService();
    }
    sendMsg(msgType: string, info: string) {
        this.msg.send(msgType, info);
    }
}
// 依赖注入
class NotificationComponent {
    constructor(msg: MessageService) {}
    sendMsg(msgType: string, info: string) {
        this.msg.send(msgType, info);
    }
}
```

