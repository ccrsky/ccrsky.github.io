---
title: react开发遇到的一些坑
date: 2020-01-03 10:00:17
tags:
  - js

category:
  - js
---

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

#### css 添加 animation 因为 css module 导致 animation name 不合法而不生效

```css
/* 定义 */
@keyframes :global(slideInFromRight) {
  0% {
    transform: translateX(100%);
  }
  100% {
    transform: translateX(0);
  }
}

/* 使用 */
.active {
  position: relative;
  transform: translateX(0);
  :global {
    animation: slideInFromRight 0.35s ease-in-out;
  }
}
```

> 因为 css module 原因，keyframes 也会像类名一样被改名，生成的 name 不合法就会导致 animation 不生效,[参考](https://github.com/css-modules/css-modules/issues/141#issuecomment-544213182)
