---
title: "了解Immutable.js"
date: 2019-05-27T00:00:00.000Z
description: "想要了解 Immutable 发现文档很牛掰，满头问号。但是为什么用了它之后能让你的应用具有更强的可读性、更好的健壮性、更少的错误以及可预测性？"
tags: ["JavaScript"]
---

想要了解 `Immutable` 发现文档很牛掰，满头问号。
但是为什么用了它之后能让你的应用具有更强的可读性、更好的健壮性、更少的错误以及可预测性？

## 了解一下

标准的 `js` 数据格式转成 `Immutable` 。
在 `js` 中，我们知道两种常见的数据类型：`Object{}` 和 `Array[]` 。`Immutable` 的思路:

- **Object{}** 变为 Map `Map({})`
- **Array[]** 变为 List `List([])`

更具体点，是使用 `Immutable` 提供的 `Map` 、`List` 和 `fromJS` 函数。

```javascript
import { Map, List, fromJS } from "immutable";
// 原生 js 数据类型
const acrejunior = {
  name: "1mu",
  pets: ["dog", "cat"]
};
// 等同于 Immutable 中的:
const immutableAcrejunior = Map({
  name: "1mu",
  pets: List(["dog", "cat"])
});
// 或者 ...
const immutableAcrejunior = fromJS(acrejunior);
```

`formJS` 是个非常有用的函数，它会把嵌套的数据结构转换成 `Immutable` 对象。它在转换的过程中会根据数据自行创建 `Maps` 和 `Lists`。

## 如何从 Immutable 变回 JS

方法简单粗暴：

```javascript
import { Map } from "immutable";
const ImmutableValue = Map({ a: "hello!" });
const ValueJS = ImmutableValue.toJS();
console.log(ValueJS);
// {a: "hello!"}
```

## 不就多做了次操作？没看出优点哇

### 从嵌套的数据结构中获取不存在的数据值

直接上代码吧。。

```javascript
const data = {
  yimu: {
    book: {
      name: "绘本"
    }
  }
};
const getData = data.yimu.book.name;
console.log(getData); // 打印 绘本
const getData2 = data.hh.book.name;
console.log(getData2);
// Uncaught TypeError: Cannot read property 'book' of undefined
// hh的属性不存在，所以报错了
```

```javascript
const data = fromJS({ yimu: { book: { name: "Will" } } });
const getData = data.getIn(["yimu", "book", "name"]);
console.log(getData); // 打印 Will
const getData2 = data.getIn(["my", "hh", "name"]);
console.log(getData2); // 打印 undefined - 但不会抛出异常
```

因为我们使用 `getIn()` 函数去获取嵌套的数据值。数据值的 key 路径不存在（换句话说，对象并非是你期望的嵌套结构），它仅仅返回 undefined 并不会抛出异常。
所以我们不用做下面这样的操作：

```javascript
if(yimu && yimu.book && yimu.book.name)
```

### 链式操作

首先，原生 js：

```javascript
const pets = ["cat", "dog"];
pets.push("goldfish");
pets.push("tortoise");
console.log(pets); // 打印 ['cat', 'dog', 'goldfish', 'tortoise'];
```

再来看看 Immutable:

```javascript
const pets = List(["cat", "dog"]);
const finalPets = pets.push("goldfish").push("tortoise");
console.log(pets.toJS()); // 打印 ['cat', 'dog'];
console.log(finalPets.toJS()); // 打印 ['cat', 'dog', 'goldfish', 'tortoise'];
```

`List.push` 返回操作之后的结果，我们可以在后面继续链式操作，而原生数组的 `push` 函数返回操作之后新数组的长度。

### 数据不可变

```javascript
const data = fromJS({ name: "Will" });
const newNameData = data.set("name", "Susie");
console.log(data.get("name")); // 打印 'Will'
console.log(newNameData.get("name")); // 打印 'Susie'
```

这是 Immutable 的核心。我们可以看到初始的 data 对象并没有改变。这就意味着当你更新它的 name 属性值为 Susie 时，并不会伴随着不可预知的行为。

这个简单的特性是非常强大的，特别是当你去搭建一些复杂的应用。

## 总结下优点

Immutable 官网所说的：

- 数据结构是可预测的
  - 数据结构 Immutable 之后，你对自己的数据结构如何操作一清二楚，我们使用 React 还是数据驱动的 Ui 框架，所以即使微小的变动，我们也不会造成一些重新渲染的问题。
- 数据操作是健壮的 Immutable 为你做了许多脏活、累活：捕获异常，提供默认值和开箱即用的创建嵌套数据结构。
- 代码可读性和简洁性强
  - Immutable 的函数式设计很困惑，但我发现，链式函数调用会让我们的代码更少、可读性更好。

## 几点注意事项

- 数据结构应该被认为是原生 js 数据结构，或者 Immutable 对象
- Immutable 对象的操作总是返回操作之后的结果
- 在 Immutable 对象上的操作并不会改变原对象，而是创建一个新对象
