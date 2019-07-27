---
title: css代码片段
date: 2019-07-27 10:06:54
tags:
  - css
  - 前端

category:
  - css
---

#### 单行文本，超出显示省略号

```css
.demo {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

#### 一行文本，超出显示省略号

```css
.demo {
  display: -webkit-box;
  /*! autoprefixer: ignore next */
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 3;
  overflow: hidden;
}
```

> 在 webpack 打包处理后，发现多上显示样式不生效是因为 autoprefix 插件默认会移除-webkit-box-orient,添加上述忽略注释即可解决。
