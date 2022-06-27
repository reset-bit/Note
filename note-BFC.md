# 前言
css渲染的时候以box作为渲染的基本单位，box类型由元素的类型和display属性决定。box类型分为`block-level box`和`inline-level box`（不包括css3）。不同类型的box参与不同类型的`formatting context`布局

`block-level box`是形式上表现为块，或display属性为block、table等的元素（不包括行块元素里的canvas/svg/input等），会生成`block-level box`，并参与`block formatting context`

`inline-level box`是display属性为inline、inline-block、inline-table的元素，会生成`inline-level box`，并参与`inline formatting context`

BFC和IFC可以理解为不同渲染区域遵循的不同规则。

# BFC
格式化上下文（`Block Formatting Content, BFC`）是Web页面的可视化CSS渲染的一部分，是块盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域。
## 产生BFC
| 元素或属性 | 属性值 |
| ---- | ----|
| 根元素body | |
| float | left/right |
| position | absolute/fixed |
| display | inline-block/flex/table-cells/table-caption |
| overflow | auto/scroll/hidden |

## BFC特性
1. 内部box在垂直方向上依次放置==》？span依旧内联行块
2. 属于同一个BFC的两个相邻box margin会发生重叠合并
3. 每个盒子的margin左边，与包含块border box的左边相接触。即使浮动也是如此
4. BFC区域不与float box重叠。计算BFC高度时，浮动元素也参与计算
5. BFC是页面上一个隔离的独立容器，容器里面的子元素不会影响到外面的元素，反之亦然

## BFC应用
1. 清除垂直方向上margin合并
2. 清除浮动高度塌陷
3. 浮动自适应左右布局

# IFC
内联格式化上下文（`inline formatting context, IFC`）

## IFC作用
1. `text-align: center;`水平居中
2. 使用某一子元素撑开父元素高度，`vertical-align: middle;`垂直居中