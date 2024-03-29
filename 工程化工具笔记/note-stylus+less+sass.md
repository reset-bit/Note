[toc]

# stylus

最简洁的css预处理器。基于缩进的语法，可省括号、分号、冒号、逗号。支持变量、内置函数。

## 安装与配置

```js
yarn add stylus stylus-loader -D// 打包文件时使用到的包
yarn remove stylus-loader -D
yarn add stylus-loader@3.0.2 -D
yarn eject// 抽取react项目配置文件，或npm run reject
// webpack.config.js
const stylRegex = /\.styl$/;// line 66
const stylModuleRegex = /\.module\.styl$/;
{// line 539
    test: stylRegex,
    use: getStyleLoaders(
        {
            importLoaders: 2,
            sourceMap: isEnvProduction && shouldUseSourceMap,
        },
        'stylus-loader'
    )
},
{
    test: stylModuleRegex,
	use: getStyleLoaders(
        {
            importLoaders: 2,
            sourceMap: isEnvProduction && shouldUseSourceMap,
        },
        'stylus-loader'
    ),
},
```

## 使用

```js
// .styl
&// 指代当前层级上级元素
// mixin混合
border-radius()
	border-radius: arguments
.button
	border-radius: 5px
```



# less

语法最接近css的预处理器。在css的语法基础之上，引入了变量，Mixin，运算以及函数等功能，降低了维护成本。less可以在多种语言、环境中使用，浏览器端、桌面客户端、服务端。

## 安装与配置

```js
yarn add less@3.13.1 less-loader@7.3.0 -D
// webpack.config.js
const lessRegex = /\.less$/;
const lessModuleRegex = /\.module\.less$/;
{
    test: lessRegex,
    exclude: lessModuleRegex,
    use: getStyleLoaders(
            {
                importLoaders: 3,
                modules: true,
                sourceMap: isEnvProduction
                        ? shouldUseSourceMap
                        : isEnvDevelopment,
            },
            'less-loader'
    ),
    sideEffects: true,
},
{
    test: lessModuleRegex,
    use: getStyleLoaders(
            {
                importLoaders: 3,
                sourceMap: isEnvProduction
                        ? shouldUseSourceMap
                        : isEnvDevelopment,
                modules: {
                        getLocalIdent: getCSSModuleLocalIdent,
                },
            },
            'less-loader'
    ),
},
```

## 使用

```js
// .less
&// 指代当前层级上级元素
@width: 10px;// 变量以@开头
// mixin混合
.box { width: 10px; }
.other { .box(); }
// 运算
@convertion-1: 5cm + 10mm;// 6cm
```



# sass

支持嵌套css、嵌套css属性，Mixin，选择器继承等。sass可省括号、分号等，scss不可省。

## 安装与配置

```js
{
    test: /\.s[ac]ss/i,
        use: [
            'style-loader',
            'css-loader',
            'sass-loader',// sass to css
        ]
}
```

## 使用

```js
// .sass
&// 指代当前层级上级元素
$min-height: 100px !default;// 变量以$开头，!default为其添加默认值
.container {
    height: $min_height;// $min_height === $min-height，下划线等同中划线
}
// 导入color.sass，相比于css import在执行时才导入并下载导致页面加载缓慢，sass在生成css时就加载指定文件
@import 'color';
// 这是一行静默注释，不会出现在转换后的css文件中
// mixin混合及传参
@mixin box($width) { width: $width; }
.other { @include box(100px); }
// 选择器继承
.error {...}
.seriousError { @extend .error }
```

