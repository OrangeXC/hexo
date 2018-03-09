---
title: JavaScript 数组中的 indexOf 方法
date: 2016-10-12 09:38:43
tags: JavaScript
---

最近项目遇到一个小问题代码我会简化成小例子展示给大家。

用心看到最后会有收获哈，基础扎实的童鞋可以直接跳到[数组类型使用](http://orangexc.xyz/2016/10/12/IndexOf-in-JavaScript-array/#Array-类型的使用)。

说到 indexOf 大家并不陌生，判断字符串是否包涵子字符串时特别常用（正则不熟练同学的利器😄）。

<!--more-->

## String 类型的使用

温习一下大家熟知的字符串用法，举个🌰

```js
let str = 'orange';

str.indexOf('o');  //0
str.indexOf('n');  //3
str.indexOf('c');  //-1
```

这里 0 和 3 分别是 o 和 n 在字符串中出现的位置。起始下标是 0。而 -1 代表未匹配。

曾经有人问我为什么偏偏是 -1 不是 null 或者 undefined。你去问制定规则的人啊！一脸无奈。

大家看到这里感觉没什么亮点啊，别急接着再来一个例子

```js
let numStr = '2016';

numStr.indexOf('2');  //0
numStr.indexOf(2);  //0
```

看到这里有个小点就是 `indexOf` 会做简单的类型转换，把数字转换成字符串 `'2'` 然后再执行。

## Number 类型的使用

大家可能会想 `number` 类型有没有 `indexOf` 方法因为会做隐式转换嘛！明确告诉大家没有，上例子

```js
let num = 2016;

num.indexOf(2);  //Uncaught TypeError: num.indexOf is not a function
```

非要对 `number` 类型使用 `indexOf` 方法嘞？那就转换成字符串咯，接着上例来写

```js
//二逼青年的写法
num = '2016';
num.indexOf(2);  //0

//普通青年的写法
num.toString().indexOf(2);  //0

//文艺青年的写法
('' + num).indexOf(2);  //0
```

第一种写法简单直接，对于已知的较短的数字也不是不可行。但是 num 变量针对不同数据是变化的时候，怎么办呢？❌

第二种写法最为常用,但对比第三种写法长了一点。哈哈，其实都可以，代码洁癖的人喜欢第三种 ✅

## Array 类型的使用

大家提起精神，大boss来了。

数组方法大家再熟悉不过了，却忽略了数组有 `indexOf` 这个方法（我个人感觉）。

干说不练瞎扯淡，遇到了什么问题，注意⚠️点又在哪里？

```js
let arr = ['orange', '2016', '2016'];

arr.indexOf('orange');  //0
arr.indexOf('o');  //-1

arr.indexOf('2016');  //1
arr.indexOf(2016);  //-1
```

这里没把例子拆的那么细，四个用例足以说明问题。

* arr.indexOf('orange') 输出 0 因为 'orange' 是数组的第 0 个元素，匹配到并返回下标。
* arr.indexOf('o') 输出 -1 因为此方法不会在每一个元素的基础上再次执行 indexOf 匹配。
* arr.indexOf('2016') 输出 1 因为此方法从头匹配直到匹配到时返回第一个数组元素的下表，而不是返回全部匹配的下标。
* arr.indexOf(2016) 输出 -1 **注意：这里不会做隐式类型转换。**

既然坑已经发现我们不妨刨根问底。去MDN官网一看究竟。对此话题感兴趣的朋友可以直接跳转到 [Array.prototype.indexOf()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/indexOf)

只想了解的朋友下面给大家官方的 Description。

> indexOf() compares searchElement to elements of the Array using strict equality (the same method used by the === or triple-equals operator).

一目了然，这里用的是严格等于（`===`）。大家做类似判断的时候多留意。不要误认为数字会转成字符串，同理字符串也不会转换成数字。

## 总结

小知识点积累，不作为深入讨论的话题，因此这里没有解释 indexOf() 的第二个参数，相信大家都知道第二个参数的作用，不知道的可以看这里[String.prototype.indexOf()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/indexOf),然后结合上面数组的链接也看一下第二个参数。
