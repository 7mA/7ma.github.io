---
title: p5.js：JavaScript创意编程程式库
tags:
  - JavaScript
  - 可视化
categories:
  - 技术笔记
date: 2021-01-23 00:00:00
permalink: p5js-beginning/
author: Kaku
---
# 1.p5.js是什么？
[**p5.js**](https://p5js.org/zh-Hans/)是一个侧重于图像编程与可视化的JavaScript库，是[Processing](https://processing.org/)的JavaScript移植版本。

![p5.js](../p5js-beginning/p5js.svg)

<!--more-->

---

# 2.开始使用p5.js

## 2.1 HTML

使用p5.js，需要先[下载JS文件](https://p5js.org/zh-Hans/download/)，或者在HTML里添加CDN链接。

```HTML
<!DOCTYPE html>
<html lang="en">
  <head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.1.9/p5.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.1.9/addons/p5.sound.min.js"></script>
    <link rel="stylesheet" type="text/css" href="style.css">
    <meta charset="utf-8" />

  </head>
  <body>
    <script src="sketch.js"></script>
  </body>
</html>
```

其中`sketch.js`为实现图像生成逻辑的JS文件。

## 2.2 JavaScript

在`sketch.js`中实现JS逻辑。

`sketch.js`的结构通常如下：

```js
function setup() {
  createCanvas(300, 300);
}

function draw() {
  background(220);
}
```

基本结构由`setup()`和`draw()`两个函数组成，作用分别如下：

- `setup()`：仅在页面初始化的时候执行，可以指定画布大小之类的初始化设置。
- `draw()`：页面初始化结束以后，每帧执行一次，用于绘制图像。

上文的示例当中，`setup()`中使用了`createCanvas(300, 300)`函数，其作用为创建一个大小为300px*300px的画布（canvas元素）。

`draw()`中使用了`background(220)`函数，其作用为指定画布背景颜色，`220`代表灰度值。

当然这个函数也支持RGB值，如下所示：

```js
// 黄色
background(255, 204, 0);

// 粉色
background('#fae');
```

因此上述代码的执行结果如下：

<p class="codepen" data-height="447" data-theme-id="light" data-default-tab="html,result" data-user="7ma" data-slug-hash="bGwydxZ" style="height: 447px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="p5js-template">
  <span>See the Pen <a href="https://codepen.io/7ma/pen/bGwydxZ">
  p5js-template</a> by Kaku (<a href="https://codepen.io/7ma">@7ma</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

---

# 3. 用法举例

## 3.1 绘制线段

可以使用`line()`来绘制线段。

首先使用`strokeWeight()`指定画笔粗细，`stroke()`指定画笔颜色。

```js
strokeWeight(4);
stroke(51); // Grayscale integer value
```

然后使用`line()`函数，`line()`可以接受4个参数，用来表示线段的起点与终点的坐标。

```js
// line(x1, y1, x2, y2)
// （30， 20）与（85， 75）
line(30, 20, 85, 75);
```

执行效果如下：

<p class="codepen" data-height="426" data-theme-id="light" data-default-tab="html,result" data-user="7ma" data-slug-hash="dypEYym" style="height: 426px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="p5js-line">
  <span>See the Pen <a href="https://codepen.io/7ma/pen/dypEYym">
  p5js-line</a> by Kaku (<a href="https://codepen.io/7ma">@7ma</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

需要注意的是，p5.js的坐标系中，**默认原点为画布左上角，x轴正方向为右，y轴正方向为下**。
所以如果不另行指定原点的话，坐标为负值的时候是无法在画布上表示出来的。

## 3.2 绘制点

可以使用`point()`来绘制点。

同样可以使用`strokeWeight()`指定画笔粗细。
`point()`可以接受2个参数，用来表示点的坐标。

```js
strokeWeight(10); // Make the points 10 pixels in size
point(30, 20);
point(85, 75);
```

结合上述线段的例子，执行效果如下：

<p class="codepen" data-height="414" data-theme-id="light" data-default-tab="html,result" data-user="7ma" data-slug-hash="oNzRjLd" style="height: 414px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="p5js-line-point">
  <span>See the Pen <a href="https://codepen.io/7ma/pen/oNzRjLd">
  p5js-line-point</a> by Kaku (<a href="https://codepen.io/7ma">@7ma</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

可以看到，后执行的`strokeWeight(10)`可以覆盖先执行的`strokeWeight(4)`的设置，同时不影响已经绘制出来的线段的粗细。

## 3.3 事件响应

p5.js中设置了各种各样的事件响应函数，可以自定义不同事件下的响应逻辑。

以拖拽事件为例，实现点随鼠标移动的逻辑。

拖拽事件响应函数为`touchMoved()`，接受`event`参数，`event`对象包含鼠标坐标等信息。

```js
// 点的初始位置
let pointX = 50;
let pointY = 50;

// ...

function draw() {
  // ...

  point(pointX, pointY);

  // ...
}

function touchMoved(event){
  pointX = event.clientX;
  pointY = event.clientY;
}
```

执行效果如下：

<p class="codepen" data-height="415" data-theme-id="light" data-default-tab="html,result" data-user="7ma" data-slug-hash="gOwJaGN" style="height: 415px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="p5js-touch-move">
  <span>See the Pen <a href="https://codepen.io/7ma/pen/gOwJaGN">
  p5js-touch-move</a> by Kaku (<a href="https://codepen.io/7ma">@7ma</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

除了事件响应函数，p5.js还有用来监听事件的信号量函数。

下面以监听键盘事件为例，实现方向键移动点的逻辑。

监听特定按键是否被按下的信号量函数为`keyIsDown()`，接受单个按键代码为参数。
可以通过[keycode.info](http://keycode.info/)获取各个按键的代码，或者使用p5.js特有的[特殊按键代码](https://p5js.org/zh-Hans/reference/#p5/keyCode)。

```js
// 点的初始位置
let pointX = 50;
let pointY = 50;

// ...

function draw() {
  // ...

  if(keyIsDown(LEFT_ARROW)){
    pointX -= 5;
  }
  if(keyIsDown(RIGHT_ARROW)){
    pointX += 5;
  }
  if(keyIsDown(UP_ARROW)){
    pointY -= 5;
  }
  if (keyIsDown(DOWN_ARROW)) {
    pointY += 5;
  }
  point(pointX, pointY);

  // ...
}
```

与上述鼠标事件示例结合以后的效果如下：

<p class="codepen" data-height="415" data-theme-id="light" data-default-tab="js,result" data-user="7ma" data-slug-hash="MWjdKyV" style="height: 415px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="p5js-key-is-down">
  <span>See the Pen <a href="https://codepen.io/7ma/pen/MWjdKyV">
  p5js-key-is-down</a> by Kaku (<a href="https://codepen.io/7ma">@7ma</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

## 3.4 数学运算

p5.js同样支持数学运算，通过[参考文档](https://p5js.org/zh-Hans/reference/)可以查看内置的数学运算函数。

内置函数确实给编码带来了方便，但是内置函数往往在运算上存在间接成本（overhead cost），所以在同样逻辑需求的前提下，要追求性能的话使用原生JavaScript的Math函数是更好的选择。

官方也发布了[原生JS与p5.js在数学运算上的性能对比结果](https://github.com/processing/p5.js-website/blob/main/src/assets/learn/performance/code/native-vs-p5/sketch.js)。

> // Results in Chrome:
//
//  p5 random took:     283.88ms.
//  Native random took: 190.01ms.
//  
//  p5 sin took:        481.14ms.
//  Native sin took:    338.33ms.
//  
//  p5 min took:        781.41ms.
//  Native min took:    538.15ms.
//  
// Results in p5 Editor:
//
//  p5 random took:     2430.28ms.
//  Native random took: 85.56ms.
//  
//  p5 sin took:        2337.90ms.
//  Native sin took:    265.94ms.
//  
//  p5 min took:        8335.63ms.
//  Native min took:    5308.00ms.

---

# 4.p5.js的优秀之处

作为一个图像编程与可视化的库，p5.js具有不少优秀的地方。

## 4.1 文档充实易查

正如官方的介绍文所言，p5.js的想定使用者十分广泛，工程师、设计师、艺术家都可以轻松上手，所以文档内容也侧重于方法导向，想用什么方法就可以随查随用，可以注意力集中于作品或者应用本身。

[参考文档](https://p5js.org/zh-Hans/reference/)的搜索功能可用性很强，而且也有不少很有创意的示例，可以帮助最大化地理解函数含义与用途。

另外，p5.js的中文文档很充实，这一点从个人角度来讲也很好用。但是目前文档支持的语言还不是很多，期待文档日后能够更加国际化。

## 4.2 轻量灵活、兼容性好

一个CDN链接，或者一个JS文件，就可以利用p5.js的所有基础功能，十分适合嵌入到其他框架中使用，与Restful API等结合使用的话，仅在前端就可以实现复杂的视觉化逻辑。

另外，预定义好渲染逻辑的话，p5.js也可以作为canvas的渲染组件使用。配合上好的设计思路的话，p5.js也可以做出来优秀的网页背景或者组件纹路。

## 4.3 官方在线编辑器

p5.js拥有官方的[在线编辑器](https://editor.p5js.org/)，无需本地环境就可以尝试各种功能。

如果登陆的话，还可以保存代码、生成分享链接等。

## 4.4 扩展功能

p5.js拥有丰富的[附加程式库](https://p5js.org/zh-Hans/libraries/)，大部分程式库为开源社群贡献。

附加程式库的使用方法与基础库相同，在HTML引用JS文件即可，但是需要在基础库（p5.js）之后。

```HTML
<!doctype html>
<html>
<head>
  <script src="p5.js"></script>
  <script src="p5.sound.js"></script>
  <script src="sketch.js"></script>
</head>
<body>
</body>
</html>
```

另外，p5.js内置WebGL模式，开启WebGL以后画布将增加z轴，可以实现3D图像。
虽然实现效果比不上three.js之类的3D特化库，但是功能也算比较充实。

---

**Reference Source:**

1. [p5.js で様々なパターンを描画してみた](https://qiita.com/Haruka-Ogawa/items/8ecb8692bf7455e7f124)
2. [p5.jsコードの最適化](https://himco.jp/2020/04/12/p5-js%E3%82%B3%E3%83%BC%E3%83%89%E3%81%AE%E6%9C%80%E9%81%A9%E5%8C%96/)
