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
// 所有服务都是由$provide服务创建的，该服务负责在运行时初始化$get()方法。$injector通过调用该方法创建服务实例。provider()负责在$providerCache中注册服务
// 假定传入函数为$get()，factory()是provider()注册服务的简略形式。如果希望在config()函数中对服务进行配置，必须用provider()定义服务
angular,module('myApp', [])
.provider('githubService', function($http) {
    let githubUrl = 'https://github.com';
    $get: function($http) { 
        self = this;
        return $http({method: self.method, url: githubUrl+'/events'}); 
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

## 装饰器

对服务进行装饰，从而进行扩展。

```js
// $delegate是最原始的服务，为了装饰其他服务，需要将其注入装饰器
let githubDecorator = function($delegate, $log) {};
angular.module('myApp')
.config(function($provide) {
    $provide.decorator('githubService', githubDecorator)
});
```

