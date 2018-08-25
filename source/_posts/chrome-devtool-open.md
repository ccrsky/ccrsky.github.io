---
title: js检测开发者工具Devtools是否打开
date: 2018-08-24 12:38:49
tags:
    - js

category: 
    - js
---


js检测开发者工具Devtools是否打开


```js
var re = /x/;
console.log(re);

re.toString = function () {
  return '您打开了控制台，想查找什么呢';
};
```
>toString() is not called on logged objects unless the console is open

[参考： https://stackoverflow.com/questions/7798748/find-out-whether-chrome-console-is-open/30638226#30638226](https://stackoverflow.com/questions/7798748/find-out-whether-chrome-console-is-open/30638226#30638226)