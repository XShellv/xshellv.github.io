---
title: JS实现链表基本操作
date: 2021-11-17
author:     "Xshellv"
catalog:    true
header-style: text
tags:
    - 链表
    - 算法
---

>本文记录日常学习中链表相关知识点。

## 链表的概念
链表是一种物理存储单元上**非连续**、**非顺序**的存储结构，数据元素的逻辑顺序是通过链表中的指针链接次序实现的。链表由一系列结点（链表中每一个元素称为结点）组成，结点可以在运行时动态生成。每个结点包括两个部分：
- 一个是存储数据元素的数据域，
- 另一个是存储下一个结点地址的指针域。

相比于线性表顺序结构，操作复杂。由于不必须按顺序存储，链表在插入的时候可以达到**O(1)**的复杂度，比另一种线性表顺序表快得多，**但是查找一个节点或者访问特定编号的节点则需要O(n)的时间**，而**线性表**和**顺序表**相应的时间复杂度分别是**O(logn)**和**O(1)**。

## JS实现简单链表增删改查

### 构造节点函数
```js
function Node(value) {
  this.value = value;
  this.next = null;
}
// 新建一个链表头节点
function LList() {
  this.head = new Node("head");
}

```

### 查找目标结点
从头部开始遍历，若找不到则移动当前结点位置，直到找到目标结点。

![单链表查找](https://cdn.xshellv.com/notes/linkedList/find_post)
```js
LList.prototype = {
  find: function (item) {
    let curr = this.head;
    while (curr && curr.value !== item) {
      curr = curr.next;
    }
    return curr;
  }
};
```

### 查找目标结点的上个结点
使用 `previous` 和 `current` 两个变量，从头部开始遍历，若找不到则移动当前结点位置，直到找到目标结点。返回`previous` 值。

![单链表查找上一个结点](https://cdn.xshellv.com/notes/linkedList/findPrev_post)
```js
LList.prototype = {
  findPrev: function (item) {
    let prev = null;
    let current = this.head;
    while (current && current.value !== item) {
      prev = current;
      current = current.next;
    }
    return prev;
  },
};
```


### 插入结点
基于 `find` 方法先找出插入位置，再从重新调整指针。

![单链表插入](https://cdn.xshellv.com/notes/linkedList/insert_post)

```js
LList.prototype = {
  insert: function (newEl, item) {
    const itemNode = this.find(item);
    if (itemNode) {
      let newNode = new Node(newEl);
      newNode.next = itemNode.next;
      itemNode.next = newNode;
    }
};
```

### 修改结点值
其实就是找到目标结点后修改其value值。

```js
LList.prototype = {
  edit: function (item, newItem) {
    const itemNode = this.find(item);
    if(itemNode){
        itemNode.value = newItem
    }
  }
};
```

### 删除结点值
和插入结点类似，先找出目标结点及其前一位结点，使前一结点的**next**指向目标结点的**next**即可。

```js
LList.prototype = {
  remove: function (item) {
    const prev = this.findPrev(item);
    if (prev) {
      const curr = prev.next;
      if (curr) {
        prev.next = curr.next;
      }
    }
  }
};
```

### 打印所有节点
从头部开始遍历输出。
```js
LList.prototype = {
  show: function () {
    let curr = this.head;
    while (curr) {
      console.log(curr.value);
      curr = curr.next;
    }
  }
};
```

## 参考文章

- [js实现链表](https://www.cnblogs.com/EganZhang/p/6594830.html)
- [百度百科链表](https://baike.baidu.com/item/%E9%93%BE%E8%A1%A8/9794473?fr=aladdin)