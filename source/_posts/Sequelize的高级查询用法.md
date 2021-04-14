---
title: Sequelize的高级查询用法
date: 2021-11-01
author:     "Xshellv"
catalog:    true
header-style: text
tags:
    - Mysql
---

> nodejs中常用的Sequelize模块能够帮助我们完成简单的sql查询，但是如果查询稍微复杂，面对蹩脚的sequelize文档我们很难完成，本编主要帮助作者记录日常摸索的复杂查询，同时也分享给各位。

## 计数统计查询并排序

### 目标

现在有如下一张sql表，现在需要将相同的name进行聚合统计，并进行降序。

**原表**：

![mysql表1](https://cdn.xshellv.com/notes/sequelize/question1)

**结果表**：

![mysql表2](https://cdn.xshellv.com/notes/sequelize/answer1)

### mysql的查询语句实现

```mysql
SELECT COUNT('name') AS `count`, `name` FROM `tags` AS `tag` GROUP BY `name` ORDER BY COUNT('name') DESC;
```

`COUNT()` 是一个特殊的函数，有**两种非常不同的作用**：它可以**统计某个列值的数量**，**也可以统计行数**。在**统计列值时要求列值非空的（不统计NULL）**。如果在COUNT()的括号中指定了列或列的表达式，**统计的就是这个表达式有值的结果数**。

### sequelize的实现写法

#### `sequelize.fn()` - 函数调用

```js
sequelize.fn(fn, args) -> Sequelize.fn
```

创建于一个相当于数据库函数的对象。该函数可用于搜索查询的`where`和`order`部分，以及做为列定义的默认值。

| 名称 | 类型 | 说明 |
| - | - | - |
| fn | String | 要调用的函数 |
| args | any | 传递给调用函数的参数 |

#### 实现

```js
const ret = await model.Tag.findAll({
    attributes: [[sequelize.fn("COUNT", "name"), "count"], "name"],
    group: "name",
    plain: false,
    raw: true,
    where,
    order: [[sequelize.fn("COUNT", "name"), "DESC"]],
    limit: body.top || 10,
  });
```

复杂的写法需要拆分理解：

```js
[sequelize.fn("COUNT", "name"), "count"]
```

上面以列 `name` 进行聚合计数，并取别名 `count` 为结果列。因此 `attributes` 的第一项为 `name` 的计数结果，以及第二项的 `name` 为查询结果。

排序这里也需要特别注意：

**错误写法**：

```js
order: [["count", "DESC"]]
```

这个表示将表的 `count` 字段进行降序，而不是聚合后的 `count` 进行排序。

**正确写法**：

```js
order: [[sequelize.fn("COUNT", "name"), "DESC"]],
```

## 参考文章

* [高性能MySQL之Count统计查询](https://blog.csdn.net/qq_15037231/article/details/81179383)
* [`seqelize.fn()` 用法介绍](https://itbilu.com/nodejs/npm/VkYIaRPz-.html#api-instance-fn)