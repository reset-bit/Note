# node基础

## 创建简易服务器

```js
npm install express express-generator -g// 添加express、express-generator至package.json dependencies依赖列表，npm5+ --save可选
npm install mysql -D

express web.tst.xxx.com
cd web.tst.xxx.com
npm install
npm start// 启动应用

// 目录解析
-bin// 配置端口号（默认3000）等
-public// 静态资源文件夹
-routes// 路由文件，mvc-controller
-views// 视图文件，mvc-view
-node_modules
-app.js// 入口文件，加载依赖包、路由等
-package.json// 记录项目所需依赖及配置信息
-package-lock.json// 记录当前状态下实际安装的包的具体来源和版本号
```

> `express`是一个基于Node.js平台，快速、开放、极简的Web开发框架。通过应用生成器工具`express-generator`可以快速创建一个应用的骨架。`express-generator `包含了 express 命令行工具。