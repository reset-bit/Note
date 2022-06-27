[toc]

# 弹性布局

`display:flex;`开启弹性布局

## 相关概念
### 弹性容器：开启了弹性布局的容器
所有标签均可设置弹性容器
开启弹性布局后，弹性容器变成块级元素，设置宽高有效，支持浮动

```js
flex-direction: row(default) | row-reverse | column | column-reverse;// 设置主轴方向

flex-wrap: nowrap(default) | wrap | wrap-reverse;// 设置弹性容器内是否允许出现多根主轴（换行），换行时不会将弹性容器宽度撑宽

flex-flow: row nowrap;// 对flex-direction flex-wrap合并设置

justify-content: flex-start(default) | flex-end | center | space-between | space-around | space-evenly;// 设置弹性项目在主轴方向上的对齐方式

align-items: stretch(default 伸展) | flex-start | flex-end | center;// 设置弹性项目在交叉轴方向上的对齐方式

align-content: stretch(default) | flex-start | flex-end | center | space-between | space-around | space-evenly;// 设置主轴（根数不限）在交叉轴方向上的对齐方式
```

### 弹性项目：弹性容器的**子元素**
成为弹性项目后，不行不块，不受`text-align`、`vertical-align`的控制，具有包裹性，设置宽高有效，可以定位但不支持浮动
```js
order: 1;// 设置弹性项目在其主轴上的摆放顺序，值为正整数，越大越排后面

align-self: inherit(default) | stretch | flex-start | flex-end | center;// 设置弹性项目在其主轴对应交叉轴方向上的对齐方式，默认听从弹性容器align-items设定

flex-grow: 0(default);// 当主轴方向上有剩余空间时，弹性项目如何瓜分剩余空间，值为自然数。显式设定主轴方向上的长度时，本属性失效

flex-shrink: 1(default);// 当主轴方向上放不下弹性项目时，弹性项目如何缩放（前提是不允许换行出现多根主轴，1意为都缩放），值为自然数。显示设定主轴方向上的长度时，本属性不受影响

flex-basis: auto(default) | 0;// 设置弹性项目以何值参与计算主轴剩余空间（0意为忽略弹性项目自身宽度，针对弹性容器本身作为剩余空间值），值为任意有效长度单位

flex: 0 1 auto | none (=== 0 0 auto) | auto (=== 1 1 auto) | x% (=== 1 1 x%) | 1/2 (=== 1/2 1 0) | n 正整数i/百分比或其他单位p (=== n i/1 auto/p);// 对flex-grow/flex-shrink/flex-basis合并设置
```

### 主轴及交叉轴
主轴：决定弹性项目在弹性容器中摆放位置和顺序，默认为水平从左向右
交叉轴：和主轴垂直的轴

> 主轴有粗细，若align-content非stretch，则主轴粗细决定于弹性容器交叉轴长度 / 主轴数量；否则主轴粗细决定于主轴上最粗的弹性项目
> 交叉轴更多的是方向的概念

# 网格布局

`display:grid/inline-gird;`开启网格/行内网格布局。

`flex`是轴线布局（一维布局），只能指定项目针对轴线的位置。`grid`是将容器划分为行和列，产生单元格，之后指定项目所在的单元格（二维布局）。

## 相关概念

### 容器：采用网格布局的区域

```js
grid-template-columns: 100px 200px | 1fr 2fr | 33.33% 66.66% | 100px auto 200px | repeat(3, 100px) | repeat(auto-fill, 100px) | minmax(100px, 1fr);// 每列列宽，fr将按比例划分总宽/；auto-fill 每一列（行）容纳尽可能多的单元格；minmax() 长度范围；
grid-template-rows: [r1] 100px [r2] 100px [r3];// 每行行高，设置网格线名称
grid-template-areas: 'header header header'
					'main . sidebar'
					'footer footer footer';// 区域命名，不需要利用区域使用.表示。区域命名会影响网格线，起始网格线自动命名为区域名-start、终止网格线自动命名为区域名-end
grid-template: ;// 对grid-template-columns/grid-template-rows/grid-template-areas合并设置

grid-auto-columns: 100px 100px;// 浏览器自动创建的多余网格的列宽，可取值同grid-template-columns
grid-auto-rows: 100px 100px;// 浏览器自动创建的的多余网格的行高
grid-auto-flow: row(default) | column | row dense | column dense;// 子元素放置顺序。后2种主要用于某些项目指定位置后，剩下的项目怎么自动放置。row dense先行后列，column dense先列后行。
grid: ;// 对grid-template-columns/grid-template-rows/grid-template-area/grid-auto-columns/grid-auto-rows/grid-auto-flow合并设置

justify-items: start | end | center | stretch;// 单元格内容水平位置
align-items: start | end | center | stretch;// 单元格内容垂直位置
place-items: start end;// 对justify-items/align-items合并设置

justify-content: start | end | center | stretch | space-around | space-between | space-evenly;// 内容区域水平位置
align-content: start | end | center | stretch | space-around | space-between | space-evenly;// 内容区域垂直位置
place-content: start end;// 对justify-content/align-content合并设置

grid-row-gap: 10px;// 行间距
grid-column-gap: 10px;// 列间距
grid-gap: 10px 10px;// 行列间距合并设置
```

### 项目：网格布局子元素

设为网格布局后，项目`float、display:inline-block、vertcal-align`等失效

```js
grid-column-start: 1 | header-start | span 2;// 项目左边框所在垂直网格线，span表示跨越多少个网格
grid-column-end: 2;// 项目右边框所在垂直网格线
grid-column: 1 / 2;// 对grid-column-start/grid-column-end合并设置

grid-row-start: 1;// 项目上边框所在水平网格线
grid-row-end: 2;// 项目下边框所在水平网格线
grid-row: 1 / 2;// 对grid-row-start/grid-row-end合并设置

grid-area: a | 1 / 1 / 3 / 3;// 指定项目放在哪一个区域

justify-self: start | end | center | stretch;// 单元格内容水平位置，跟justify-item用法一直，但只作用于单个项目
align-self: start | end | center | stretch;// 单元格内容垂直位置，类align-self
place-self: start start;// 对justify-self/align-self合并
```

### 行列、单元格、网格线

行：容器里的水平区域

列：容器里的垂直区域

单元格：行列交叉区域

网格线：划分网格的线。正常情况下，n行有n+1根水平网格线，m列有m+1根垂直网格线