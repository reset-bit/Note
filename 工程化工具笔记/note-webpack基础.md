[toc]

# webpack基础

webpack是一种前端资源构建工具，一个静态模块打包器。

webpack基于`node.js`平台，模块化采用`commonJS`语法。默认只能处理`js or JSON`文件。

```js
development// 开发环境
production// 生产环境（经过代码压缩）

npm init// 初始化项目，需要填写webpack项目名称等信息
webpack ./src/index.js -o ./build/built.js --mode=development[production]// 打包
webpack// 运行，有文件输出
npx webpack-dev-server// 运行webpack-dev-server，无文件输出（只在内存中编译打包）
```

## 全局安装与本地安装

全局安装下，包安装在`node`安装目录下的`node_modules`文件夹中，可以供命令行使用。

本地安装可以让每个项目拥有独立的包，直接通过require()方法去引用模块，保证不同版本包之间的互相依赖。

不推荐仅全局安装：需要手动配置包路径；对于包的更新不好管理。

```js
npm i webpack webpack-cli -g// 全局安装
npm i webpack webpack-cli -D// 本地安装
```

## 核心概念

### entry

指示webpack以哪个文件为入口起点开始打包，分析构建内部依赖图

### output

指示webpack打包后资源bundles输出到哪里、如何命名

### loader

令webpack处理非`js/JSON`文件

### plugins

令webpack执行范围更广的任务，如优化、压缩、重定义环境中变量等

### mode

设置一系列环境变量，令webpack切换相应工作环境

```js
npm i css-loader style-loader less less-loader -D// 样式加载loader
npm i html-webpack-plugin -D// 复制模板html并自动注入资源
npm i file-loader url-loader html-loader -D// 图片加载loader

// webpack.config.js webpack配置文件
const { resolve } = require('path');// 解构resolve用来处理绝对路径
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'built.js',
        path: resolve(__dirname, 'build');
    },
    module: {
        rules: [
            {
                test: /\.css$/,// 正则表达式匹配文件后缀
                // 执行顺序：从右到左、从上到下
                use: [
                    'style-loader',// 创建style标签，插入head中生效
                    'css-loader'// 将css文件变成commonJS模块加载js中，内容是样式字符串
                ]
            },
            {
                test: /\.less$/,
                use: [
                    'style-loader',
                    'css-loader',
                    'less-loader'// 将less转为css
                ]
            },
            // 图片处理
            {
                // 无法处理html中加载的img图片
                test: /\.(jpg|png|gif)$/,
                loader: 'url-loader',
                options: {
                    // 图片大小小于8kb，将转为base64编码，不显示为图片
                    limit: 8 * 1024,
                    // url-loader默认使用es6模块化解析，html-loader解析时会出现[objecy Module]问题，此选项可关闭url-loader es6模块化，使用commonJS解析
                    esModule: false,
                    // 图片重命名：取hash前10位+文件原扩展名
                    name: '[hash:10].[ext]'
                },
                type: 'javascript/auto'// webpack5弃用url-loader、file-loader
            },
            {
                // 引入html中img，从而能被url-loader处理
                test: /\.html$/,
                loader: 'html-loader'
            },
            {
                // 处理除了css/js/html后缀名的文件
                exclude: /\.(css|js|html)$/,
                loader: 'file-loader'
            }
        ]
    },
    plugins: [
        // 复制模板HTML，并自动引入打包输入的所有资源
        new HtmlWebpackPlugin({
            template: './src/index.html'
        })
    ],
    mode: 'development',
    // webpack 热加载 自动编译
    devSever: {
        contentBase: resolve(__dirname, 'build'),// 项目构件后的路径
        compress: true,// 启用gzip压缩
        port: 3000,
        open: true,// 自动打开浏览器
    }
};
```

## 

