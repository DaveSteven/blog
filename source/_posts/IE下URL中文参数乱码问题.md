---
title: IE下URL中文参数乱码问题
date: 2018-06-06 19:24:21
categories:
- Front-End
tags:
- Javascript
- encodeURI
---

### 问题排查
公司的B端项目在做IE兼容性处理，发现高德地图在IE无法显示。
经排查，是因为接口URL携带的中文参数在IE中显示为乱码，服务端返回了错误信息。

### 解决办法
使用[encodeURI()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURI)方法进行处理，将中文参数进行编码即可。
```Javascript
const url = 'http://www.xxxx.com';
const params = `?key=xxxx&${encodeURI(city)}`

this.http.get(url + params);
```