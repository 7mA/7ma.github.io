---
title: YouTube嵌入式播放器优化
tags:
  - 前端
  - 优化
categories:
  - 技术笔记
author: Kaku
date: 2022-02-01 13:34:00
permalink: youtube-embed-optimization/
---
# 0.前言

将YouTube嵌入页面，只需要将官方提供的嵌入代码复制粘贴就可以了。
但是要想让嵌入播放器的渲染和布局与页面的契合性更好，就需要进一步的优化措施。
下面列举几个最近导入的实例。

![YouTube](/youtube-embed-optimization/yt-icon.png)

<!--more-->

---

# 1.嵌入视频宽度自适应

YouTube的分享按钮里面都有一个网页嵌入的选项，提供视频嵌入组件的HTML代码：

```html
<iframe width="560" height="315" src="https://www.youtube.com/embed/2SVQ1LYElnY" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
```

但是可以看到，像这种只有一个`<iframe>`标签的HTML，是没有自适应大小或者响应式的功能的。像YouTube提供的代码中就预定义了组件的高度与宽度。
但是由于不同设备的分辨率和尺寸不同，所以显示效果是无法预测的。
比如上面YouTube的`width="560" height="315"`这个尺寸，在PC的网页端没有什么问题，但是在移动端就会出现下面这种情况：

![移动端布局崩坏](/youtube-embed-optimization/mobile-ng.png)

原本的页面宽度被定宽的嵌入组件强行撑开，页面布局出现了崩坏。

针对这个问题，如果能让视频嵌入组件的宽度自适应页面的话就可以解决了，但是怎么才能做到呢？

如果只是宽度的话，把宽度设置成`width="100%"`就行了，但是高度也需要按比例设置才可以，否则就会这样：

![视频宽高比异常](/youtube-embed-optimization/mobile-ng-2.png)

宽度本身是在布局确定之后计算得出的，在这之前是没办法定义高度的，所以这条路子是行不通的。

为了解决这个问题，可以给嵌入组件定义一个容器，给容器设置如下样式：

```css
.video-container {
  position: relative;
  padding-bottom: 56.25%;
  /* 16:9 */
  padding-top: 25px;
  height: 0;
}
```

这样可以让容器适应父层元素的宽度。
`padding-bottom: 56.25%;`为容器设置了56.25%宽度的下内边距，以保证在填充嵌入组件以后播放器16：9的宽高比。
`padding-top: 25px;`可以让播放器的视频画面上下留有黑色边缘，如果取消的话视频画面的上下边缘会直接与页面背景相连。

然后再给嵌入组件添加如下样式：

```css
.video-container iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
```
让播放器按照预定的16:9比例充满父层容器，这样以后嵌入播放器的大小就可以适应屏幕的尺寸了。

```html
<div class="video-container">
  <iframe src="https://www.youtube.com/embed/2SVQ1LYElnY" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>
```

<div class="video-container">
  <iframe src="https://www.youtube.com/embed/2SVQ1LYElnY" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

其实最开始在自己考虑对策的时候，已经自己添加了自定义的CSS，但是在测试的时候才发现Even主题里已经[集成相关的样式](https://github.com/ahonn/hexo-theme-even/blob/master/source/css/_base.scss#L87-L102)了，所以这里就直接沿用了。

---


# 2.缩略图模拟播放器元素实现延迟加载
