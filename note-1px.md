# 响应式布局中的1px问题

台式机物理像素：逻辑像素=1：1；手机端n：1

`dpr: devicePiexlRatio`设备像素比。台式机为1，iphone6/7/8为2，占4个实际像素。

目的：使逻辑1px真正显示到物理设备上的1px。

解决方案：css3媒体查询

```css
.border-1px { position: relative; }
.border-1px:after {
    content: '';
    postion: absolute;
    left: 0;
    top: 0;
    box-sizing: border-box;
    border: 1px solid black;
    transform-orign: left top;
    z-index: -1;
}
@media screen and (-webkit-min-device-pixel-ratio: 1) {
    .border-1px:after {
        width: 100%;
        height: 100%;
        transform: scale(1);
    }
}
@media screen and (-webkit-min-device-pixel-ratio: 2) {
    .border-1px:after {
        width: 200%;
        height: 200%;
        transform: scale(0.5);
    }
}
@media screen and (-webkit-min-device-pixel-ratio: 3) {
    .border-1px:after {
        width: 300%;
        height: 300%;
        transform: scale(0.333333);
    }
}
```

