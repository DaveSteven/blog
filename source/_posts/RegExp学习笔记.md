---
title: RegExp学习笔记
date: 2018-08-14 11:12:40
categories:
- Front-End
tags: 
- Javascript
---
自己在正则表达式方面一直很薄弱，平时工作中感觉也很少用到，其实是自己没有掌握，想不到用而已。当用到的时候，在百度copy一下，并没有去深入研究过。为了提升自己，为了以后更好的机会，巩固JS基础是刻不容缓的，所以先从正则表达式开始学习。花了大概一天的时间，结合慕课网的视频，把正则的知识点整理了一下。

首先介绍一款很实用的图形工具：[regexper-static](https://github.com/javallone/regexper-static)。该工具能够直观的展示一个正则表达式是如何进行匹配的，来看一下匹配邮箱的正则转成图形如何表示：
```Javascript
const re = /^[0-9a-zA-Z._]+@[0-9a-zA-Z-]+(\.[a-z]+)+$/;
```
<img src="/images/regexp_email.png" style="display: inline-block !important">
是不是很神奇？分析起来也很容易，这款工具极大的提高了我学习正则的效率。接下来进入正题：
## 1、RegExp 对象
RegExp 是 JavaScript 内置对象。
有两种方法可以实例化 RegExp 对象：
#### 字面量
创建的时候，需要有两个`/`包围：
```Javascript
const re = /is/;  // 匹配is
```
#### 构造函数
可以传两个参数，第一个为表达式，第二个为修饰符，修饰符接下来会讲。
```Javascript
const re = new RegExp('is');  // 匹配is
```
我们可以用创建的正则表达式做一些事情，比如说把 `is` 替换为 `IS` ：
```Javascript
const re = /is/g;
let str = 'This is a apple';
let ret = str.replace(re, 'IS');
console.log(ret);  // => ThIS IS a apple
```
我们可以看到 `str` 中所有的 `is` 都替换成了 `IS` ，那么如果我只想匹配 `is` 这个单词，并不想匹配包含 `is` 的单词，该如何做？可以加入 `\b` 单词边界进行匹配：
```Javascript
const re = /\bis\b/g;
let str = 'This is a apple';
let ret = str.replace(re, 'IS');
console.log(ret);  // => This IS a apple
```
`\b` 是用来匹配一个字边界，即字与空格间的位置。

### 修饰符
在 JavaScript 中，常用的修饰符有：
#### i，不区分（ignore）大小写
例：匹配 `de` ， 替换为 `X`
```Javascript
const re = /de/ig;
const str = 'abcDe';
let ret = str.replace(re, 'X');
console.log(ret);  // => abcX
```
#### g，全局（global）匹配
例：匹配所有的 `_` ，替换为 `仙女`
```Javascript
const re = /_/g;
const str = '从前有座_山，_山上住着小_';
let ret = str.replace(re, '仙女');
console.log(ret);  // => 从前有座仙女山，仙女山上住着小仙女
```
#### m，多（more）行匹配