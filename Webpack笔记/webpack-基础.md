# webpack 5.x

静态打包工具，默认只能打包js。

开发环境能够识别es6语法，生产环境在此之上进行代码压缩、规范化单元测试、代码检查等。

```js
npm i -y
npm i webpack webpack-cli -D
npx webpack ./src/main.js --mode=development// webpack.config.js未存在
npx webpack// webpack.config.js存在
```

