---
title: 通过POST请求下载Excel
date: 2018-07-25 11:15:40
categories:
- Front-End
tags:
- Javascript
- Work
---
我在公司负责维护一个B端的后台管理系统，大部分的数据导出都是通过拼接url（域名 + api地址 + 参数）打开浏览器新窗口来实现下载的。
但是目前出现了一个问题，产品说我要导出所有种类的数据，但是无法下载，以下是Chrome的报错信息：
![](/images/page_not_work.jpg)
经排查，是URL过长导致的。URL的长度大概为7500+，已经远远超过了Chrome浏览器URL限制范围。我在网上找到了其他浏览器对URL字符的长度限制：
- Firefox，长度限制为65,000 个字符
- Safari， 长度限制为80,000 个字符
- Opera，长度限制为190,000 个字符

我们来解决一下这个问题。此项目为Vue单页应用，使用axios进行ajax请求。在网上也找到了一些解决方案，基本的思路如下（非IE浏览器）：
1. 创建 `post` 请求，拿到数据。
2. 创建 [Blob](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob) 对象。
3. 建立 `a` 标签，设置为隐藏。
4. 建立文件链接，添加到 `a` 标签上。
5. 将 `a` 标签添加到Dom节点中。
6. 模拟点击事件，下载。

思路清楚之后，代码就很简单了，贴代码：

```Javascript
// 1. post请求
axios.post({
  url: 'xxx/downloadExcel',
  method: 'post',
  data: {
    ids
  },
  responseType: 'arraybuffer'
}).then(res => {
  const filename = 'excel.xls';
  downloadExcel(res, filename);
})
```
<!-- more -->
在请求的这一步，axios中有个responseType属性，主要是用来设置服务端返回的数据类型，默认为 `json` ，我们这里设置为 [arraybuffer](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)。
为了方便以后使用，我把下载Excel的方法封装起来了。请求成功后，执行 `downloadExcel()` 方法，来看一下 `downloadExcel()` 代码：
```Javascript
export const downloadExcel = (data, filename) => {
  const blob = new Blob([data]);  // 创建blob对象

  if ('download' in document.createElement('a')) {  // 非IE下载，只有 Firefox 和 Chrome 支持 download 属性
    const fileLink = URL.createObjectURL(blob);  // 会产生一个类似blob:d3958f5c-0777-0845-9dcf-2cb28783acaf 这样的URL字符串
    const eLink = document.createElement('a');
    eLink.download = filename;
    eLink.style.display = 'none';
    eLink.href = fileLink;
    document.body.appendChild(eLink);
    eLink.click();
    URL.revokeObjectURL(eLink.href);  // 释放URL对象
    document.body.removeChild(eLink);
  } else {
    navigator.msSaveBlob(blob, filename);
  }
}
```
首先来做一下浏览器兼容，判断 `a` 标签中是否有 `download` 属性。只有 Firefox 和 Chrome 支持 `download` 属性，该属性规定被下载的超链接目标。所以将 `filename` 文件名赋值给 `eLink.download` 。接下来关键的一步，创建文件的 URL 对象，我们使用 [URL.createObjectURL()](https://developer.mozilla.org/zh-CN/docs/Web/API/URL/createObjectURL#Browser_compatibility) 方法进行创建，创建后就可以将 URL 给到 `eLink.href` 。然后把 `elink` 添加到 Dom 节点，触发点击事件就可以下载了。注意一点，下载完毕后一定要把 `eLink` 移除，并且释放创建的 URL 对象。因为每次调用 `createObjectURL()` 方法时，都会创建一个新的 URL 对象，即使已经用相同的对象作为参数创建过。

#### 总结
1. axios 中的 `responseType` 属性可以设置服务器返回的数据类型，默认为 `json` ，还有 `arraybuffer` 、`blob` 、 `document` 、 `text` 、 `stream` 5种类型。
2. `arraybuffer` 对象用来表示通用的、固定长度的原始二进制数据缓冲区。不能直接操作，要通过类型数组对象或 DataView 对象来操作，它们会将缓冲区中的数据表示为特定的格式，并通过这些格式来读写缓冲区的内容。
3. `a` 超链接对象中的 `download` 属性只有 Firefox 和 Chrome 支持，该属性规定被下载的超链接目标，实际上就是文件名。
4. `Blob` 对象表示一个不可变、原始数据的类文件对象。
5. `URL.createObjectURL()` 方法可以创建一个 URL 对象，参数为 `blob` ，用来创建 URL 的 File 对象或者 Blob 对象。可以用来显示图片，播放音乐，下载等等。大致长这个样子：`blob:http://www.demo.com/9779127a-3bc1-4b2d-9d0e-79d8f327481d`。为了获得最佳性能和内存使用状况，当不需要 URL 对象的时候，可以调用 `URL.revokeObjectURL()` 方法来释放该对象。