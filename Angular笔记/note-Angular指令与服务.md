[toc]

# 指令

本质上是Angular扩展的具有自定义功能的HTML元素的途径。

```js
// 双向绑定
<div>
    <label>thire url:</label>
    <input type="text" ng-model="theirUrl" />
    <div my-directive some-attr="theirUrl" my-link-text="google"></div>// 为指令传递数据
</div>
angular.module('myApp', [])
	// 指令定义
    .directive('myDirective', function () {
        return {
            /*  声明创建指令
                <my-directive></my-directive>// E
                <div my-directive my-url='http://www.google.com'></div>// A
                <div class='my-directive'></div>// C
                <!-- directive:my-directive expression -->// M            
            */
            retrict: 'A',
            replace: true,// 只留下template内容
            
            // false(default) | true | {}
            // .. | 从父级作用域继承并创建一个新的作用域对象 | 隔离作用域，无法访问外部作用域
            scope: {
                // 绑定策略
                myUrl: '=someAttr',// 本地作用域属性与父级作用域上属性进行双向数据绑定
                myLinkText: '@',// 本地作用域属性与同DOM属性的值进行绑定
                myOtherText: '&'// 生成指向父级作用域的包装函数
            },
            
            // HTML文本 | function(tElement, tAttrs){return str;} 
            template: `
                <div>
                    <label>my url</label>
                    <input type='text' ng-model='myUrl'/>
                    <a href='{{myUrl}}'>{{myLinkText}}</a>
                </div>
            `,
            templateUrl: url,// 一个代表外部HTML文件的字符串 | function(tElement, tAttrs){return str;}
            transclude: true,// 嵌入，一旦设置必须为true，用于创建可复用的组件
            controllerAs: 'myController',// 控制器别名，用以发布控制器
            
            // str | []
            // 包含?前缀，若在当前指令中没有找到所需要的控制器，会将null作为传给link函数的第四个参数
            // 包含^前缀，会在上游指令链中查找需要的控制器
            // 以上两种前缀
            require: 'ngModel', 
            
            // 编译函数，负责对模板DOM进行转换。如果设置了complie函数，说明希望在指令和实时数据被放到DOM之前进行DOM操作，在这个函数中进行DOM操作是安全的
            compile: function(tEle, tAttrs, transcludeFn) {
                ...
                return function(scope, ele, a) {
                    // this is link function
                };
            },
                
            // 链接函数，创建可以操作DOM的指令。当同时定义compile与link函数时，complie会重载link。someContorller取决于require。
            link: function(scope, ele, attrs[, someController]) {},
        };
    });
```

## 内置指令

```jsx
// 布尔属性
ng-disabled// 支持按条件禁用，HTML只会检查disabled是否存在
ng-readonly
ng-checked
ng-selected
// 类布尔属性
ng-href// 使用当前作用域属性动态创建URL时，应用ng-href代替href。Angular将会等到插值完成后生效
ng-src// 在ng-src对应表达式生效之前不要加载图像
// 具备子作用域
ng-app// 任何具有ng-app属性的DOM元素将被标记为$rootScope的起始点
ng-controller// 为嵌套在其中的指令创建一个子作用域。控制器应当尽可能简单，不能将$scope赋值为值类型
ng-include// 加载、编译并包含同域的外部HTML片段到当前应用中
ng-switch ng-switch-when ng-switch-default on='propertyName'
ng-view// 设置将被路由管理和放置在HTML中的视图的位置
ng-cloak// 直到路由调用页面才显示元素（懒加载）
ng-if
ng-show ng-hide// 通过添加或移除ng-hide css类实现
ng-select ng-options// 与<select>进行绑定
ng-repeat// 支持$index $first $last $middle $even $odd
{{}}
ng-bind// 使用ng-bind手动实现{{}}：<p ng-bind='hello'></p>。解决含有{{}}元素不立刻渲染，导致未渲染内容闪烁FOUC问题
ng-model// 将input/select/text area等控件与数据进行绑定
ng-change// 在表单输入发生变化时计算给定表达式的值
ng-form ng-submit ng-click// 在表单内部另外嵌套表单，原本HTML不允许表单嵌套
ng-class// 动态设置class
ng-attr-(suffix)// 手动添加标签属性
// 部分指令使用示例
// ng-switch
<input type='text' ng-model='person.name'></input>
<div  ng-switch on='person.name'>
	<p ng-switch-default>The winner is </p>
    <h1 ng-switch-when='Air'>{{person.name}}</h1>
</div>

// ng-select
<div ng-controller='CityController'>
	<select ng-model='city' ng-options='city.name for city in cites'>
    	<option value=''>choose city</option>
    </select>
    best city: {{city.name}}
</div>
angular.module('myApp', [])
.controller('CityController', function($scope) {
    $scope.cites = [
        {name:'beijing'},
        {name:'qingdao'}
    ];
});
```



# 服务

单例对象，提供了一种能够在应用的整个生命周期内保持数据的方法，能够在控制器之间进行通信，并保证数据一致性。

```js
angular.module('myApp.Services', [])
/**
* @params str 服务名称
* @params function | [] 服务工厂函数，在整个应用生命周期内只调用一次
*/
.factory('githubService', function($http) {// 注入其他服务
    let githubUrl = '...';
    let runUserRequest = function() {};
    // 支持返回简单类型、函数、对象等任意类型数据
    return {
        // 将方法设置为服务对象的属性来暴露给外部
        events: function(...) { return runUserRequest(); }
    };
});
```

## 创建服务

```js
// factory
angular.module('myApp')
.factory('githubService', ['$http', function($'http') {
    // 返回对象
    return {
        getUserEvents: function() {}
    };
}])

// service
// 注册一个支持构造函数的服务
let Person = function() {}
angular.service('personService', Person);

// provider
// 所有服务都是由$provide服务创建的（是provider的语法糖），该服务负责在运行时初始化$get()方法。$injector通过调用该方法创建服务实例。provider()负责在$providerCache中注册服务
// 如果希望在config()函数中对服务进行配置，必须用provider()定义服务
angular,module('myApp', [])
.provider('githubService', function($http) {
    let githubUrl = 'https://github.com';
    $get: function($http) { 
        self = this;
        return $http({method: self.method, url: githubUrl+'/events'}); // githubService服务实例
    }
})
.config(function(githubServiceProvider) {
    githubServiceProvider.xxx();
});

// constant
// 将已经存在的变量值注册为服务，并将其注入到应用的其他部分中。通常用于配置数据
angular.module('myApp').constant('apiKey', '123');// 需要注册的常量名字与值
.controller('MyController', function($scope, apikey) {})

// value
// 适用于返回$get()返回常量的情形，通常用于注册服务对象或函数
angular.module('myApp').value('apikey', '123');
```

> 模块提前配置-config：
>
> angular模块可以使用config在被加载和执行之前对模块自身进行配置，只有提供者provider和常亮constant才能注入到config中

## 装饰器

对服务进行装饰，从而进行扩展。

```js
// $delegate是最原始的服务，为了装饰其他服务，需要将其注入装饰器
let githubDecorator = function($delegate, $log) {};
angular.module('myApp')
.config(function($provider) {
    $provider.decorator('githubService', githubDecorator)
});
```
