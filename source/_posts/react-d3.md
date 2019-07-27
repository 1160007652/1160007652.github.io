---
title: D3 数据可视化
abstract: '欢迎来到我的博客, 这是一篇加密文章。'
message: 请输入密码后按回车键
date: 2018-11-25 14:39:54
tags:
    - react
    - d3
    - 可视化
copyright:
password:
---

### 资源
#### 代码：https://github.com/1160007652/react-d3
#### Demo：https://1160007652.github.io/react-d3

-----

### 简介

D3 (或者叫 D3.js )是一个基于 web 标准的 JavaScript 可视化库. D3 可以借助 SVG, Canvas 以及 HTML 将你的数据生动的展现出来. D3 结合了强大的可视化交互技术以及数据驱动 DOM 的技术结合起来, 让你可以借助于现代浏览器的强大功能自由的对数据进行可视化.

<!-- more -->

### svg

SVG 是使用 XML 来描述二维图形和绘图程序的语言。

### svg 速学

#### 标签

> rect 矩型 \ circle 圆 \ ellipse 椭圆 \ line 直线 \ polygon 多边形 \ polyline 曲线 \ path 路径\ text 文字

#### 属性

##### 共有属性
> stroke 描边颜色 \ stroke-width 描边宽度
> stroke-linecap 线条两端的样式 : butt 直角原状态 \ round 圆角状态 \ square 加长方角状态
> stroke-dasharray 蚂蚁线
> stroke-opacity 描边透明度 [ 0 - 1 ]

> fill 填充颜色
> fill-opacity 填充透明度 [ 0 - 1 ]

> css-opacity 整个元素的透明度 [ 0 - 1 ]

##### rect 属性
> x y : 起始点位置
> rx ry : 可使矩形产生圆角
> width : 宽带
> height: 高度

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
  <rect x="50" y="20" rx="20" ry="20" width="150" height="150"
  style="fill:red;stroke:black;stroke-width:5;opacity:0.5"/>
</svg>
```

##### circle 属性
> cx cy : 圆形的位置
> r : 半径

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
  <circle cx="100" cy="50" r="40" stroke="black"
  stroke-width="2" fill="red"/>
</svg>
```

##### ellipse 属性
> cx cy : 椭圆的位置
> rx ry : 水平半径 垂直半径

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
  <ellipse cx="300" cy="80" rx="100" ry="50"
  style="fill:yellow;stroke:purple;stroke-width:2"/>
</svg>
```

##### line 属性
> x1 y1 : 起始坐标
> x2 y2 : 结束坐标

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
  <line x1="0" y1="0" x2="200" y2="200"
  style="stroke:rgb(255,0,0);stroke-width:2"/>
</svg>
```

##### polygon 属性
> points : 置入坐标点

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
  <polygon points="220,10 300,210 170,250 123,234"
  style="fill:lime;stroke:purple;stroke-width:1"/>
</svg>
```

##### polyline 属性
> points : 置入坐标点

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
  <polyline points="20,20 40,25 60,40 80,120 120,140 200,180"
  style="fill:none;stroke:black;stroke-width:3" />
</svg>
```

##### path 属性

路径规则：
M = moveto(M X,Y) ：将画笔移动到指定的坐标位置
L = lineto(L X,Y) ：画直线到指定的坐标位置
H = horizontal lineto(H X)：画水平线到指定的X坐标位置
V = vertical lineto(V Y)：画垂直线到指定的Y坐标位置
C = curveto(C X1,Y1,X2,Y2,ENDX,ENDY)：三次贝赛曲线
S = smooth curveto(S X2,Y2,ENDX,ENDY)
Q = quadratic Belzier curve(Q X,Y,ENDX,ENDY)：二次贝赛曲线
T = smooth quadratic Belzier curveto(T ENDX,ENDY)：映射
A = elliptical Arc(A RX,RY,XROTATION,FLAG1,FLAG2,X,Y)：弧线
Z = closepath()：关闭路径

> d : 存放路径规则

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
    <path d="M150 0 L75 200 L225 200 Z" />
</svg>
```
##### text 属性

1、直接写出文字

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
  <text x="0" y="15" fill="red">I love SVG</text>
</svg>
```

2、路径写文字

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1"
xmlns:xlink="http://www.w3.org/1999/xlink">
   <defs>
    <path id="path1" d="M75,20 a1,1 0 0,0 100,0" />
  </defs>
  <text x="10" y="100" style="fill:red;">
    <textPath xlink:href="#path1">I love SVG I love SVG</textPath>
  </text>
</svg>
```
3、多行文本

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
  <text x="10" y="20" style="fill:red;">Several lines:
    <tspan x="10" y="45">First line</tspan>
    <tspan x="10" y="70">Second line</tspan>
  </text>
</svg>
```

3、文本点击跳转

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1"
xmlns:xlink="http://www.w3.org/1999/xlink">
  <a xlink:href="http://www.w3schools.com/svg/" target="_blank">
    <text x="0" y="15" fill="red">I love SVG</text>
  </a>
</svg>
```

##### 章节小结

在学习 **D3.js** 时，我先进行了 **svg**  的了解。快速的学习了**svg**的标签及属性。

### D3构造-大黄人

使用到的**D3.js** Api:
- select 选中元素
- arc 构造弧型
- append 添加新的svg元素
- attr 设置属性
- transition 动画
- duration 时间

[点击获取 **完整版** 代码](https://github.com/1160007652/react-d3/blob/master/src/page/example-1-emot/index.js)
[demo 演示](https://1160007652.github.io/react-d3) 进入点击菜单 **大黄人**

``` js
import React from 'react';
import { select, arc} from 'd3';

class EmotSVG extends React.Component {
    constructor(props) {
        super(props);
        this.refSvgDom = React.createRef();
        this.emotStyle = {
            emotWidth: 900,
            emotHeight: 500
        }
    }
    componentDidMount() {
        this.svg = select(this.refSvgDom.current);
        this.draw();
    }
    draw () {
        // ......
    }
    render() {
        const { emotWidth, emotHeight } = this.emotStyle;
        return (
            <div className="example-emot">
                <svg  ref={this.refSvgDom} width={emotWidth} height={emotHeight} ></svg>
            </div>
        );
    }
}

export default EmotSVG;
```

在画这个大黄人时，遇到的问题

1、 选择dom

> 使用React 提供的 **React.createRef()** APi, 获取Dom, 并且将Dom提供给D3使用。

2、把绘制大黄人的方法抽离

> 提供 draw() 绘制方法。