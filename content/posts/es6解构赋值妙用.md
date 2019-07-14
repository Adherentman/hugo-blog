---
title: "Es6解构赋值妙用"
date: 2019-07-14T18:02:08+08:00
description: "ES6的解构赋值详解"
tags: ["JavaScript"]
---

平常我们都这样去操作对象：

```javascript
let options = {
  isPush: true,
  isOpen: false
}

let isPush = options.isPush;
let isOpen = options.isOpen;
```

## 对象解构

```javascript
let {isPush, isOpen} = options;
console.log(isPush)		// true
console.log(isOpen)		// false
```

## 解构赋值

当额外定义一个局部变量`isClick`，但是在`options`上没这个对应名称的属性，所以会返回`underfined`。

```javscript
let {isPush, isOpen, isClick} = options;
console.log(isPush) // true
console.log(isOpen)    // false
console.log(isClick)  // underfined
```

但是我们可以给他初始值

```javscript
let {isPush, isOpen, isClick = false} = options;
console.log(isPush)		// true
console.log(isOpen)		// false
console.log(isClick)		// false
```

当然我们可以给未定义的变量赋初始值，那也就可以给变量改名：

```javascript
let {isPush: isUnPush, isOpen: isUnOpen, isClick: isUnClick = false} = options;
console.log(isUnPush)			// true
console.log(isUnOpen)			// false
console.log(isUnClick)		// false
```

我们还可以发现当`isClick`不存在这个变量的时候，我们可以边给变量名再赋初始值；

## 嵌套对象解构

```javascript
let options = {
	name: "Music",
	status: "open",
	musicData: {
		link: {
			img: "xxxx.com",
			audio: "xxxxxx.com"
		},
		info: {
			time: "2018-09-26"
		}
	}
};

let { musicData: { info } } = options;
console.log(info);		// {time: "2018-09-26}
```

与之前提到的给变量改名：

```javascript
let options = {
	name: "Music",
	status: "open",
	musicData: {
		link: {
			img: "xxxx.com",
			audio: "xxxxxx.com"
		},
		info: {
			time: "2018-09-26"
		}
	}
};

let { musicData: { info: createTime } } = options;
console.log(createTime);		// {time: "2018-09-26}
```

所以但我们在读数据之后为了防止`underfined`所以我们都去做`&&`但是写多了也很烦。趁之前*嵌套对象结构*则可以解决这个问题。

## 数组解构
```javascript
let aninmal = ["dog", "cat", "duck"];

let [, dier, disan] = aninmal;

console.log(dier);		// cat
console.log(disan);		// duck
```
这样我们还可以不要第一个，只要后面2个值。

## 数组解构
数组解构和对象解构很像，可以给*默认值*，可以*嵌套数组解构*但是还有一点特殊地方就是可以*交换两个变量的值*.

```javascript
let x = 1,
	  y = 3;

[x, y] = [y, x]

console.log(x)			// 3
console.log(y)			// 1
```


### 不定元素
```javascript
let colors = ["red", "blue", "green"];

let [firstColor, ...anyColors] = colors;

console.log(firstColor);		// "red"
console.log(anyColors);		// ["blue", "green"]
```

## 混合解构

```javascript
let options = {
	name: "Music",
	status: "open",
	musicData: {
		link: {
			img: "xxxx.com",
			audio: "xxxxxx.com"
		},
		info: {
			time: "2018-09-26"
		},
		bigArray: [1, 2, 3, 4]
	}
};

let { musicData: {link}, bigArray: [a] } = options;
console.log(link.img);		// "xxxx.com"
console.log(a);				// 1
```

这种方法很有效，当我们从JSON配置中提取信息的时候，不在需要遍历整个解构了。