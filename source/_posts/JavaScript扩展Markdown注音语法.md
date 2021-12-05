---
title: JavaScript实现Markdown风格注音语法
tags:
  - JavaScript
categories:
  - 技术笔记
author: Kaku
date: 2021-12-05 19:44:38
permalink: javascript-ruby-grammar-extension/
---

# 0.原生Markdown实现注音的方式

原生的Markdown没有注音语法，如果想在Markdown中实现注音的话只能借助HTML的`<ruby>`标签。

```HTML
<ruby>被注音文字<rt>注音内容</rt></ruby>
```

实现效果如下：

> <ruby>被注音文字<rt>注音内容</rt></ruby>

但是这种表示方式不仅麻烦，也不符合Markdown“所见即所得”的风格。所以下面借由JavaScript来实现Markdown风格的注音语法。

<!--more-->

---

# 1.定义注音语法格式

可以模仿Markdown的超链接语法定义注音语法如下：

```Markdown
\{被注音文字\}\(注音内容\)
```

---

# 2.JavaScript模拟语法实现

语法定义好了，下面只需要通过JavaScript来规定从语法到最终显示结果（HTML）之间的正则表达匹配与映射就可以了。

```JavaScript
function markdownRubyExtension(str) {
    return str.replace(/\{([^{}()]+)\}\(([^{}()]+)\)/g, function(match, $1, $2) {
        return '<ruby>' + $1 + '<rt>' + $2 + '</rt></ruby>'
    });
}
```

其中`str`用于指定需要执行语法映射的文字，此处对所有正文适用。

```JavaScript
let posts = document.getElementsByClassName('post-content');

for(let post of posts) {
  post.innerHTML = markdownRubyExtension(post.innerHTML);
}
```

---

# 3.结果示例

> {吃}(chī){葡萄}(pú táo){不}(bù){吐}(tǔ){葡萄}(pú táo){皮}(pí)，{不}(bù){吃}(chī){葡萄}(pú táo){倒}(dào){吐}(tǔ){葡萄}(pú táo){皮}(pí)。（[Markdown原文](https://github.com/7mA/7ma.github.io/blob/master/source/_posts/JavaScript%E6%89%A9%E5%B1%95Markdown%E6%B3%A8%E9%9F%B3%E8%AF%AD%E6%B3%95.md?plain=1#L66)）

> {東京}(とうきょう){特許}(とっきょ){許可}(きょか){局長}(きょくちょう){今日}(きょう){急遽}(きゅうきょ){休暇}(きゅうか){許可}(きょか){拒否}(きょひ)。（[Markdown原文](https://github.com/7mA/7ma.github.io/blob/master/source/_posts/JavaScript%E6%89%A9%E5%B1%95Markdown%E6%B3%A8%E9%9F%B3%E8%AF%AD%E6%B3%95.md?plain=1#L68)）

> 간장 {공장}(工場) {공장장}(工場長)은 {강}(薑) {공장장}(工場長)이고 된장 {공장}(工場) {공장장}(工場長)은 {장}(張) {공장장}(工場長)이다.（[Markdown原文](https://github.com/7mA/7ma.github.io/blob/master/source/_posts/JavaScript%E6%89%A9%E5%B1%95Markdown%E6%B3%A8%E9%9F%B3%E8%AF%AD%E6%B3%95.md?plain=1#L70)）

---

**Links:**

1. [JS中使用正则表达式g模式和非g模式的区别_任磊-CSDN博客](https://blog.csdn.net/sinat_36728518/article/details/106146310)
2. [各種小説投稿サイトのルビ記法をJavaScriptで実現する - Qiita](https://qiita.com/8amjp/items/d7c46d9dee0da4d530ef)
