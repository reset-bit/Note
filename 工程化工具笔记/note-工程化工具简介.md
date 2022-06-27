[toc]

# 工程化工具

## node
单线程，基于chrome v8的js运行环境。可以快速编写js服务器。
node_cache存放缓存模块
node_gobal存放全局安装模块

## webpack
webpack是一个前端资源加载/打包工具，能够支持程序员模块化编程。它根据模块的依赖关系进行静态分析，然后将这些模块按照指定的规则生成对应的静态资源。
webpack本身只能处理javascript模块，如果要处理其他类型的文件，需要使用loader进行转换。如需要在应用中添加css文件，需要使用到css-loader和style-loader。前者遍历css文件，找到url表达式处理；后者将原来的css代码插入页面style标签中。

webpack-dev-sever是webpack上层开发服务，内置静态资源web服务器，以监听模式自动运行，热（实时）加载，通过一个socket.io服务实时监听变化。

## npm/npx
npm是在安装node.js时附带的包管理器。npm script是npm内置的一个功能，允许在package.json文件里使用scripts字段定义任务。该字段的每一个属性对应一段脚本，通过shell运行脚本命令。

npx从npm5.2版本开始与其捆绑在一起，执行时首先检查路径中是否存在要执行的包。如果不存在就先安装后再执行它，执行完毕后将清除该包。
```js
// 配置全局安装模块位置
npm config get prefix
npm config set prefix 'xxx'
// 查看全局安装过的模块
npm list -g --depth=0// 只显示第一层级的模块
// 全局安装
npm install vue-cli -g
npm install xxx -D/npm install xxx --save-dev// 安装到devDependcies
// 初始化项目
vue init webpack project_name// 项目名全小写，不可以数字开头

npm init// 初始化npm项目，创建package.json文件
npm init -y// 初始化npm项目，过程中所有选项默认yes

npm install xxx/npm i xxx// 找到当前目录下package.json中dependencies/devDependencies需要的依赖包批量安装
npm i xxx --save/npm i xxx -S// 局部安装

npm start// webpack找到依赖相关包并打包（npm install），webpack-dev-sever打开默认8080内置服务器，热加载呈现项目
npm run dev// 运行package.json中配置的dev节点
npm run server// 运行package.json中配置的server节点
npm run build// 打包成为物理文件
npm run eject// 解压相关依赖，不可逆过程
```
## yarn
解决npm前期无法管理相关依赖包的问题。
```js
npm install -g yarn
yarn global dir
yarn config set global-folder 'E:\Node\yarn_global'
yarn cache dir
yarn config set cache-folder 'E:\Node\yarn_cache'
yarn config get registry
yarn config set registry https://registry.npm.taobao.org -g
yarn global add create-react-app// 已不建议使用全局安装
create-react-app -h// 此处配置环境变量
yarn eject// 显示配置文件，显示后不可隐藏，不可回退

yarn init// 初始化项目
yarn start// 启动项目
yarn add
yarn remove
yarn install// 安装项目依赖
yarn build// 打包项目，使用webpack管理工具，在js中导入图片等需要打包后显示。package.json中配置“homepage":"./"。在build目录下开启本地服务器即可

// yarn : 无法加载文件 D:\Software\Node\node_global\yarn.ps1，因为在此系统上禁止运行脚本。主要原因是powershell的执行策略阻止了此次操作。
get-ExecutionPolicy// Restricted表示执行受限
set-ExecutionPolicy -Scope CurrentUser
RemoveSigned
get-ExecutionPolicy// 
```

## grunt/glup
grunt与npm类似，有大量现成的插件封装了常见的任务。优点是灵活，缺点是集成度不高，要写很多配置，无法做到开箱即用。
gulp是基于流的自动化构建工具，处理可以管理和执行任务，还支持监听文件、读写文件。优点是可以单独完成构建，也可以和其他工具搭配使用。缺点与grunt类似。