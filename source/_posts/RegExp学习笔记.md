---
title: RegExp学习笔记
date: 2018-08-14 11:12:40
categories:
- Front-End
tags: 
- Javascript
---
自己在正则表达式方面一直很薄弱，平时工作中感觉也很少用到，其实是自己没有掌握想不到用而已。当用到的时候，在百度copy一下，并没有去深入研究过。为了提升自己，为了以后更好的机会，巩固JS基础是刻不容缓的，所以先从正则表达式开始学习。花了大概一天的时间，结合慕课网的视频，把正则的知识点整理了一下。

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
<!-- more -->
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
若存在换行 `\n` 并且有开始 `^` 或结束 `$` 符的情况下，和 `g` 一起使用实现全局匹配，因为存在换行时默认会把换行符作为一个字符进行匹配。`g` 只匹配第一行，加上 `m` 后实现多行，每个换行符之后就是开始。
```Javascript
const str = 'abcdefg\nabcdefg';
const re = /abc/gm;
let ret = str.replace(re, 'ABC');
console.log(ret);  // ABCdefg\nABCdefg
```

## 2、元字符
元字符包括：
- 原义文本字符
- 元字符

元字符使正则表达式具有处理能力。
在正则表达式中有特殊含义的非字母字符：
`* + ? $ ^ . | \ () {}`

还有以下几种：

| 字符 | 含义                        |
| ---- | --------------------------- |
| \t   | 水平制表符                  |
| \v   | 垂直制表符                  |
| \n   | 换行符                      |
| \r   | 回车符                      |
| \0   | 空字符                      |
| \f   | 换页符                      |
| \cX  | 与X对应的控制字符(Ctrl + X) |

## 4、字符类
元字符 `[]` ，使用 `[]` 来构建一个简单的类，所谓类是指符合某些特性的对象，一个泛指。
例：`[abc]` ，匹配字符a或b或c的内容
```Javascript
const str = 'a1b2c3d4';
let ret = str.replace(/[abc]/g, 'X');  // => X1X2X3d4
```
表达式 `[abc]` 把a或b或c自定义归为一类，然后匹配这类的字符。
转成图形表示：
<img src="/images/regexp_abc.png" style="display: inline-block !important">

## 5、范围类
例：匹配所有字符，[a-z]
```Javascript
const str = 'a1b2c3d4';
let ret = str.replace(/[a-z]/g, '*');  // => *1*2*3*4
```
转成图形表示：
<img src="/images/regexp_a-z.png" style="display: inline-block !important">

## 6、预定义类

| 字符 | 等价类 | 含义 |
| ---- | ------- | ---------------- |
| . | [^\r\n] | 除了回车和换行符之外的所有字符 |
| \d | [0-9] | 数字字符 |
| \D | [^0-9] | 非数字字符 |
| \s | [\t\n\x0B\f\r] | 空白符 |
| \S | [^\t\n\x0B\f\r] | 非空白符 |
| \w | [a-zA-Z_0-9] | 单词字符（字母、数字、下划线） |
| \W | [^a-zA-Z_0-9] | 非单词字符 |

例：匹配一个ab + 数字 + 任意字符的字符串
```Javascript
const re = /ab[0-9][^\r\n]/;
```
如果不使用预定义类，写起来比较麻烦。
转成图形表示：
<img src="/images/regexp_ab0-9rn.png" style="display: inline-block !important">
再看一下使用预定义类之后：
```Javascript
const re = /ab\d./;
```
转成图形表示：
<img src="/images/regexp_ab0-9rn2.png" style="display: inline-block !important">

#### 边界匹配字符

| 字符 | 含义 |
| -- | -- |
| ^ | 以xxx开始 |
| $ | 以xxx结束 |
| \b | 单词边界 |
| \B | 非单词边界 |

先看一下 `\b` 单词边界，上代码：
```Javascript
const str = 'This is a girl';
let ret = str.replace(/is/g, '0');
console.log(ret);  // => Th0 0 a girl
```
我们看到 `This` 中的 `is` 也被替换成了 `0` ，这不是我们希望的结果，加上 `\b` 看一下：
```Javascript
const str = 'This is a girl';
let ret = str.replace(/\bis\b/g, '0');
console.log(ret);  // => This 0 a girl
```
加上单词边界，可以精确匹配 `is` 这个单词。
转成图形表示：
<img src="/images/regexp_word-boundary.png" style="display: inline-block !important">
如果只想匹配 `This` 中的 `is` 该怎么做？看一下此 `is` 左侧是没有空格的，右侧有空格，可以这样写：
```Javascript
const str = 'This is a girl';
let ret = str.replace(/\Bis\b/g, '0');
console.log(ret);  // => Th0 is a girl
```
`is` 左侧的匹配改为 `\B` 非单词边界即可。

#### `^` 和 `$`
例：匹配@ + 任意字符
```Javascript
const str = '@Product@Develop';
let ret = str.replace(/@./g, '哈');
console.log(ret);  // => 哈roduct哈evelop
```
符合条件的 `@P` 、`@D` 都被替换成了 `哈` 。如果只想匹配开头的字符，需要在前面加上 `^` ：
```Javascript
const str = '@Product@Develop';
let ret = str.replace(/^@./g, '哈');
console.log(ret);  // => 哈roduct@develop
```
那么如果只想匹配结束的字符，则需要加上 `$` ：
```Javascript
const str = '@Product@Develop';
let ret = str.replace(/@.$/g, '哈');
console.log(ret);  // => @Product@Develop
```
可以看到字符是原样输出的，因为结尾处并没有匹配到合适的字符。那改一下匹配的规则：任意字符 + @
```Javascript
const str = '@Product@Develop@';
let ret = str.replace(/.@$/g, '哈');
console.log(ret);  // => @Product@Develo哈
```
`p@` 被替换为了 `哈` 。

## 6、量词

| 字符 | 含义 |
| -- | ------ |
| ? | 出现`0`次或`1`次（最多出现`1`次） |
| + | 出现`1`次或`多`次（至少出现`1`次） |
| * | 出现`0`次或`多`次（任意次） |
| {n} | 出现`n`次 |
| {n, m} | 出现`n`到`m`次 |
| {n,} | 至少出现`n`次 |

量词在正则表达式中频繁用到，需要记住。使用方法会在下面记录。

## 7、贪婪模式和非贪婪模式
例子：匹配一个字符3-6次，并替换成字符 `哈`
```Javascript
const str = '12345678';
let ret = str.replace(/\d{3,6}/, '哈');
console.log(ret);  // 哈78
```
`123456` 被替换为了 `哈` ，说明匹配了6次。为什么不匹配3次就停止呢？这就是贪婪模式，在匹配成功的前提下，尽可能多的匹配。
再看一下非贪婪模式：
```Javascript
const str = '12345678';
let ret = str.replace(/\d{3,6}?/, '哈');
console.log(ret);  // 哈45678
```
只需要在量词后加上 `?` ，就变成了非贪婪模式。可以看到 `123` 被替换为了 `哈` ，当满足了3个字符的匹配之后，不会再进行匹配。

## 8、分组
例：匹配Byron单词3次
我想了想，很简单啊，加上量词就好了：
```Javascript
const re = /Byron{3}/;
```
但是我转成图形之后一看，好像不太对。
转成图形表示：
<img src="/images/regexp_group.png" style="display: inline-block !important">
只有字符 `n` 循环匹配了3次，并不是整个单词进行匹配。这时候就需要用到分组 `()` 了。
```Javascript
const re = /(Byron){3}/;
```
只需要给 `Byron` 加上 `()` 就可以匹配整个单词了。
转成图形表示：
<img src="/images/regexp_group2.png" style="display: inline-block !important">
再来看个例子：
匹配字母+数字连续出现3次的场景，替换为 `哈`
```Javascript
const str = 'a1b2c3d4';
let ret = str.replace(/([a-z]\d){3}/g, '哈');
console.log(ret); // => 哈d4
```

#### 或 `|` 使用方法
使用 `|` 可以做多个分支的匹配，比如：
```Javascript
const str = 'ByronCasper';
let ret = str.replace(/Byron|Casper/g, '哈');
console.log(ret);  // => 哈哈
```
转成图形表示：
<img src="/images/regexp_or.png" style="display: inline-block !important">
#### 反向引用
例：将2018-08-13转换成08/13/2018。
```Javascript
const date = '2018-08-13';
let ret = date.replace(/(\d{4})-(\d{2})-(\d{2})/, '$2/$3/$1');
console.log(ret);  // => 08/13/2018
```
可以看到匹配的每一项都用 `()` 进行了分组，3个分组匹配到的内容分别是 `$1：2018` `$2：08` `$3：13` ，匹配到之后便可使用 `$1 $2 $3` 对内容进行处理。
转成图形看会更清楚：
<img src="/images/regexp_group3.png" style="display: inline-block !important">
图例中的group #1...group #3就是$1...$3，用来代表其中的内容。

#### 忽略分组
圆括号会有一个副作用，使相关的匹配会被缓存，此时可用 `?:` 放在第一个选项前来消除这种副作用。
```Javascript
const re = /(?:\d{4})-(\d{2})-(\d{2})/;
```
转成图形表示：
<img src="/images/regexp_group4.png" style="display: inline-block !important">

## 9、前瞻
- 正则表达式从文本头部向尾部开始解析，文本尾部方向称为“前”，头部方向称为“后”；
- 前瞻就是在正则表达式匹配到规则的时候，向前检查是否符合断言；
- JavaScript不支持后顾；
- 符合特定断言的称为 肯定/正向 匹配，不符合的称为 否定/负向 匹配；

| 名称 | 正则 | 含义 |
| ---- | ------ | -------- |
| 正向前瞻 | exp(?=assert) |  |
| 负向前瞻 | exp(?!=assert) |  |
| 正向后顾 | exp(?<=assert) | JavaScript不支持 |
| 负向后顾 | exp(?<!assert)| JavaScript不支持 |

例：正向前瞻，匹配后面是数字的字符，替换为 `哈`
```Javascript
const str = 'a1*2b3';
let ret = str.replace(/\w(?=\d)/g, '哈');
console.log(ret);  // => 哈1*2哈3
```
实际上就是把数字前面的字符替换为了目标字符。
转换成图形表示：
<img src="/images/regexp_lookahead.png" style="display: inline-block !important">
再来看负向前瞻：
```Javascript
const str = 'a1*2b3';
let ret = str.replace(/\w(?!\d)/g, '哈');
console.log(ret);  // => a哈*哈b哈
```
该项正则表达式是将不符合条件的字符转换成目标字符。`1 2 3` 都为数字，不符合条件。