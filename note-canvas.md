# canvas

默认有宽高：`width:300px; height: 150px;`，自定义宽高只能设置内联样式

```html
<canvas id='canvas' width='400' height='400'>您的浏览器不支持canvas</canvas>

<script>
    let canvas = document.getElementById('canvas');
    if(canvas.getContext) {
        let ctx = canvas.getContext('2d');
        ctx.moveTo();// 移动上下文
        ctx.stroke();// 描边
        ctx.fill();// 填充

        // 矩形
        ctx.rect();
        ctx.strokeRect(pos1, pos2, pos3, pos4);
        ctx.fillRect(lx, ly, width, height);
        // 弧
        ctx.arc(x, y, r, beginDeg, endDeg);// Math.PI=180deg，以水平方向为0deg

        ctx.beginPath();
        ctx.closePath();

        // 坐标系
        ctx.translate(x, y);
        ctx.rotate(Math.PI);
        ctx.save();
        ctx.restore();// 复位坐标系，会消耗save()

        // 文字-总参考x轴
        ctx.fillText(content, x, y);
        ctx.strokeText(content, x, y);
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';
        ctx.font = '12px Consolas';
        // 其它
        ctx.lineWidth = 5;// 线宽
        ctx.lineCap = 'butt' | 'round' | 'square';// 线条末端形状：平头、圆头、方头
        ctx.lineJoin = 'round' | 'bevel' | 'miter';// 线条相交方式：圆交、斜交、斜接
        ctx.strokeStyle = '#fff';// 描边
        ctx.fillStyle = '#fff';// 填充色
    }
</script>
```

