[toc]
# 综合
## font-size=0
消除`display: inline-block|inline;`因代码换行产生的空格（缝）。默认`font-size: 16px;`

## 样式继承
多数边框类属性，如padding\margin\background\border，都不能被继承

## sticky-footer
页脚相对于body作绝对定位；设置body最小高度，保证页脚不会覆盖内容。适用于页面内容较少的页脚

## 文本省略
```css
//单行文本
{
    white-spacing: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}
//多行文本
{
    overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
}
```

## 盒模型
简述盒模型：
- 标准盒模型：box-sizing:content-box。是默认值。标准盒子模型 ＝ margin + border + padding + width | height（width | height = content）；（IE盒）怪异盒模型：box-sizing:border-box。IE盒子模型 ＝ margin + width | height（width | height = border + padding + content）

- display决定盒子类型。block；inline；inline-block

  > IE5.5为IE盒模型，IE6/7/8未添加doctype则使用IE盒模型，IE9以后为W3C模型

百分比总结：
- `width/height`参考父级宽高
- `margin/padding`参考父级宽度
- `font-size`参考父级字体大小
- `line-height`参考自身字体大小
- `background-postion`参考自身宽高-背景宽高
- `background-size`参考父级宽高
- 绝对定位：`width/left/right`参考参照块宽度；`height/top/bottom`参考参照块高度
> 子元素`margin\padding`百分比参考父元素width。可制作宽度为百分比+固定比例的盒子。但不能直接放内容。需要内部再套一层盒子并绝对定位

## 浏览器内核及私有前缀

私有前缀：浏览器对已经足够成熟的css新属性的提前支持。浏览器总是先支持前缀css3，之后支持正常css3。

`-moz`：火狐`FireFox Gecko`内核

`-ms`：IE内核

`-webkit`：`webkit`内核，常见的是谷歌、苹果以及各移动端

`-o`：`Opera`内核

## px/em/rem

px：相对长度单位，相对于显示器屏幕分辨率而言。

em：相对长度单位，相对于当前对象内（父级元素）文本的字体尺寸，会继承父级元素的字体大小。任意浏览器默认字体高都是16px，所有未经调整的浏览器`1em=16px`。

> 在body中声明，font-size=62.5%，令em值变为10px。这时只需要将原来的px除10，然后换上em作为单位就可以啦。如`12px=1.2em`

rem：相对长度单位，相对于html根元素。

需要适配各种移动设备，使用rem。


# 属性举例

## overflow
visiable(defult) | hidden | scroll(常设滚动条) | auto(不超出无滚动条)，可隐式表达边界

## writing-mode
书写模式，horizontal-tb(defult) | vertical-lr | vertical-rl，可解决li Z字型摆放问题。控制行级元素，具有继承性，并且使当前盒子具有包裹性

## object-fit
指定可替换元素的内容应该如何适应到其使用的高度和宽度确定的框。可替换元素的展现效果不是由css控制的，这些元素是一种外部对象。典型的可替换元素有`<iframe>/<video>/<img>`等。

## transition
过渡，不可使用数字衡量的属性无法应用transition属性
分写：`transition-property | transition-duration | transtion-delay | transition-time-function`
> 加在hover上没有鼠标移出动画

## box-shadow
水平方向阴影偏移量、垂直方向阴影偏移量、阴影羽化距离、阴影扩散距离（扩散过程中颜色不会变淡）、阴影颜色
> 羽化即模糊，光的衍射：从黑到白过渡。
> 设置某一方向阴影+将阴影扩散距离设置为负值==>设置某一方向上的阴影

## animation

```css
animation: animation-name animation-duration animation-timing-function 
			animation-delay animation-iteration-count 
			animation-direction animation-fill-mode

animation-duration: 动画时长，秒或毫秒计
animation-timing-function: 动画速度曲线 linear | ease | ease-in | ease-out | ease-in-out | cubic-bezier(n,n,n,n)
animation-delay: 动画延时
animation-iteration-count: 动画播放次数 n | infinite
animation-direction: 是否轮流反向播放动画 nomal | alternate
animation-fill-mode: 动画结束后表现 none | forwards

/* 使用perspective开启透视效果，仅作用于元素后代 */
perspective: 800px;
perspective-origin: 50% | left | bottom 50%...;
transform: rotateX(0deg) | rotateY(0deg) | rotateZ(0deg);
/* 使用transform-style开启3D模式 */
/* flat子元素不保留其3D位置；perserve-3d保留其3D位置 */
transform-style: flat(default) | preserve-3d;
```

## 超链接a相关

```css
a:link { color: #06F; text-decoration: none; }/* 未访问的链接 */
a:visited { color: #999; text-decoration: line-through; } /* 已访问的链接 */
a:hover { color: #F00; text-decoration: underline; }/* 鼠标移动到链接上 */
a:active { color: #F0F; } /* 选定的链接 */
```

> a:hover 必须位于 a:link 和 a:visited 之后
> a:active 必须位于 a:hover 之后
