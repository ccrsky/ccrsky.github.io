---
title: UTF-8 without BOM
date: 2017-04-21 11:49:42
tags:
 - 常见问题
 - js
 
category: 
    - 常见问题
---

今天遇到一个文件编码问题(非法字符`\ufeff`)，导致代码异常；经过排查发现：文件编码是 UTF-8 + BOM，表现形式如下：
```js
var a = '*{name:'test'}';
```
但是在编辑器（我用的`sublime text`）中**看不到那个星号**，在chrome源码显示面板中可以看到，于是设置文件编码`sublime text : File > Set File Encoding to(或者保存utf-8编码)`，星号依然存在。

解决方案：
由于本机安装了`Android studio`,于是用AS打开该文件，设置编码文件为`GBK` ,然后设置为`UTF-8`(特殊字符会变成`??`去掉多余的`?`),即可去掉BOM。

在排除过程中也发现另外一个类似的问题:服务端json编码`UTF-8 + BOM`导致json解析报错。
解决方案：
设置服务端数据编码格式为`UTF-8`