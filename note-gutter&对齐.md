[toc]
# 关于HTML标签宽高的记录
## 外标签宽高固定，由外向内写
内层标签使用百分比实现宽高设置。注意每一层标签都要设置百分比。多见于内容区。
> 想要添加子标签自动适应，可通过设置标签布局（如`text-align:justify`）实现

## 内标签宽高固定，由内向外写
外标签不需要设置宽高，由内部标签自动撑高。多见于ul导航栏。
> 当需要添加子标签时，能够自动适应。
> ul导航栏：li标签内a标签使用padding或margin根据【鼠标滑过】实际效果判断。li不需要设置百分比宽度，支持添加a标签自适应

# 可调宽度的gutter--CSS变量
```html
<div class="style-show">
    <ul>
        <li><a href=""></a></li>
        <li><a href=""></a></li>
        <li><a href=""></a></li>
        <li><a href=""></a></li>
    </ul>
</div>
```
```css
:root {
	--gutter: 10px;
}
/* 由内向外撑高标签 */
.style-show {
	overflow: hidden;
}
.style-show ul {
	margin-right: calc(-1 * var(--gutter));
	font-size: 0;
}
.style-show li {
	box-sizing: border-box;
	width: 25%;
	display: inline-block;
	padding-right: var(--gutter);
}
.style-show a {
	display: block;
	height: 250px;
	background-color: brown;
}
```
基本思路：
	缝的设置方式优先级：`padding>border>margin`，margin因不受盒模型控制、垂直方向合并与穿透等原因，优先级最低。
	从直观上讲，为每个子项正常设置缝。利用margin将最右或最底盒子拉出一个gutter的距离。使用CSS变量实现可变宽度的gutter。
	外套两层标签，由外到内称层1与层2。层1负责控制实际显示区域，需设置`overflow:hidden;`。若由外向内写，需要在层1处限制宽高。层2负责将实际显示区域拓宽一个gutter的距离，即将指定方向上的margin设置为`-1*gutter`宽度。层2内子项可依照需求、按照优先级选择gutter的实现方式。

# 文本布局
## 垂直居中
### 行级元素 inline\inline-block
`line-height:[height]px;`
### 块级元素 block
设置`:before/after`伪类，令其`height=100%;`。在伪类及需要对齐的标签中使用`vertical:middle;`实现。

## 水平居中
### 行块级\块级元素 inline-block\block
`text-align:center;`
> 行内联元素宽高总由包裹的内容决定

## 水平拉伸
设置`:after`伪类，令其`width=100%;`。在父标签中使用`text-align:justify;`实现。

> `text-align:justify;`仅对多行元素有效！
> `text-align-last: justify;`设置最后一行文本在被强制换行之前的对齐规则，是继承属性 

## 水平、垂直居中
### 绝对定位
【宽高已固定的】子元素相对父元素绝对定位，并设置：
```css
top:0;
bottom:0;
left:0;
right:0;
margin:auto;
```
### 绝对定位+transform
【宽高已固定的】子元素相对父元素绝对定位，并设置：
```css
top:50%;/* 父元素50% */
left:50%;
transfrom:translate(-50%, -50%);/* 子元素50% */
```
