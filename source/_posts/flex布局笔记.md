---
title: flex布局笔记
date: 2021-02-24
author:     "Xshellv"
catalog:    true
header-img: "post-bg-js-version.jpg"
tags:
    - css
---

> 用于记录flex布局的常用知识点。

## 内容溢出的解决方法
```html
<div class="main">
    <img alt="" class="logo" src="pic.jpg">
    <p class="notice">This is long long content.</p>
</div>
```

```css
.main {
    display: flex;
}
.logo {
    width: 100px;
    height: 100px;
    margin: 10px;
}
.notice {
    flex: 1; /* 剩余分配比例 */
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}
```

当 `.notice` 很长时，我们想到的是设置`text-overflow: ellipsis`  `white-space: nowrap` `overflow: hidden`，但是省略符根本没有出现且内容溢出，所以必须要解决这个问题。

### 方法一

设置 `width: 0` 即可：
```css
.notice {
    flex: 1;
    width: 0;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}
```
如果不设置宽度，.content可以被子节点无限撑开；因此.notice总有足够的宽度在一行内显示所有文本，也就不能触发截断省略的效果。

### 方法二
设置 ` overflow: hidden`即可：

```css
.content {
    flex: 1;
    overflow: hidden；
}
```

## 参考文章

- [flex布局中，保持内容不超出容器的解决办法](https://blog.csdn.net/zgh0711/article/details/78270555)
